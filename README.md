# Install Ubuntu Budgie 19.04/19.10 on Schneider SCT101CTM

This document describes the process to install Ubuntu Budgie 19.04 on Schneider SCT101CTM. It is required a working installation of Ubuntu (or virtual machine) to prepare the bootable USB. Installation of regular Ubuntu 19.04 is similar but it is slow with 2GB RAM. The same process applies for 19.10 versions

## Prepare bootable USB on a Ubuntu installation

Download Ubuntu/Ubuntu Budgie

Install isorespin. This tablet requires a 32bit UEFI boot to load a 64bits OS. Ubuntu comes with 64bits bootloader. To replace with a 32bit UEFI bootloader use [isorespin](http://linuxiumcomau.blogspot.com/2017/06/customizing-ubuntu-isos-documentation.html) script

Create the iso with 32bit bootloader

`isorespin.sh -i ubuntu-budgie-19.04-desktop-amd64.iso`

Create a bootable USB with Startup Disk Creator

## Change UEFI boot order in Windows on Schneider

Boot Windows 10 and insert bootable USB

Open a terminal window with cmd and reboot into UEFI

`shutdown /r /fw`

In Ubuntu from command line: `systemctl reboot --firmware-setup`

Or after shutdown press *Power on* and *Volume+* until Schneider logo appears. From grub menu select *System Setup*

In UEFI menu go to *Boot* tab and change the boot order or override boot to the USB, then Save & exit

## Install Ubuntu

Install Ubuntu Budgie from grub menu. If you select *Try before installing* the mouse pointer position will be inverted  from screen which makes it difficult to use. This is related to the x session manager. Regular Ubuntu comes with gdm3 whereas Budgie comes with lightdm. gdm3 3.32+ has this partially solved: start session with the screen in horizontal. The pointer is not rotated and you will be able to rotate the screen after. Though starting the session in vertical, the problem persist

If screen orientation is inverted execute `xrandr -o 1`

For network connection you can use wifi or usb tethering. If using wifi better connect now as in the next step you will need it and the mouse pointer will be rotated (if installing Ubuntu Budgie)

Alternatively you can connnect to wifi using nmcli

`nmcli d wifi connect <wifiSSID> password <wifiPassword>`

It is recommended the installation option that deletes the entire SSD as it is only 32GB, barely enough for dual boot, altohugh is doable (not covered here)

## Fix pointer rotation

After installation at first boot the mouse pointer is inverted from the screen (only Ubuntu Budgie). To fix this install gdm3. If wifi is not available Shutdown and Power on (no reboot)

Press Ctrl+Alt+T to open terminal

Rotate screen

`xrandr -o 1`

Install gdm3
```
sudo apt-get update
sudo apt-get install gdm3
```
In the configuration menu select gdm3


## Fix screen orientation

To fix screen orientation create a udev rule for the accelerometer sensor in /etc/udev/hwdb.d/61-sensor-local.hwdb
```
sensor:modalias:acpi:BOSC0200*:dmi:bvnAmericanMegatrendsInc.:bvrSCH12i.WJ210Z.KtBJRCA03*
 ACCEL_MOUNT_MATRIX=-1, 0, 0; 0, 1, 0; 0, 0, 1
```
(second line has to be indented)

Update udev rules:
```
sudo systemd-hwdb update
sudo udevadm trigger -v -p DEVNAME=/dev/iio:device0
```
Shutdown and power on


## Touchscreen driver

Wifi, bluetooth and audio works out of the box. For the touchscreen it is needed to install the driver and firmware. Thanks to [gsl-firmware](https://github.com/onitake/gsl-firmware) I've ported the firmware to linux. Use files in [touchscreen](touchscreen) folder. There are two open source drivers: gslx680_ts_acpi and silead_ts

### gslx680_ts_acpi (the easy way)

Copy [silead_ts.fw](touchscreen/silead_ts.fw) to /lib/firmware

Compile the driver [gslx680_ts_acpi](https://github.com/onitake/gslx680-acpi) to obtain gslx680_ts_acpi.ko. Test the module with `sudo insmod gslx680_ts_acpi.ko`. You need to recompile for different kernels

To do this permanent:

Create the file */etc/modules-load.d/gslx680_ts_acpi.conf* and add:

`gslx680_ts_acpi`

Then copy the module to the system folder and build dependencies:
```
sudo cp gslx680_ts_acpi.ko /lib/modules/$(uname -r)/
sudo depmod
```
To test the installed module:
```
sudo modprobe gslx680_ts_acpi
lsmod | grep gslx680_ts_acpi
```
### Kernel driver silead_ts (the hard way recommended)

Delete file */etc/modules-load.d/gslx680_ts_acpi.conf* if created

Copy [kernel firmware](touchscreen/gsl1680-schneider-sct101ctm.fw) to */usr/lib/firmware/silead/*

General instructions to compile the kernel [here](https://kernel-team.pages.debian.net/kernel-handbook/ch-common-tasks.html#s-common-official)

Add sources. Uncomment lines in /etc/apt/sources.list

```
deb-src http://archive.ubuntu.com/ubuntu disco main
deb-src http://archive.ubuntu.com/ubuntu disco-updates main
```
Prepare environment

`sudo apt-get install build-essential devscripts equivs libncurses5 libncurses5-dev`

Build dependencies with mk-build-deps so later they can be easily removed

`mk-build-deps linux --install --root-cmd sudo --remove`

Download sources

`apt-get source linux`

Change to sources folder (e.g.: linux-5.3.0)

Apply [patch](patches/touchscreen_dmi.patch)

`patch -p1 < <path to patch>/touchscreen_dmi.patch`

Some SD cards are not initialized. If SD card is not initalized with error *mmc2: error -84 whilst initialising SD card* apply [patch](patches/sdhci.patch)

`patch -p1 < <path to patch>/sdhci.patch`

Copy kernel .config

`cp /boot/config-$(uname -r) .config`

Change config options
```
scripts/config --disable DEBUG_INFO
scripts/config --set-str CONFIG_LOCALVERSION "-touchscreen"
```
(Optionally you can do make `make localmodconfig` and add additional drivers with `make menuconfig` to reduce building time and increase free ram)

Compile and install
```
make -j 4
sudo make modules_install
sudo make install
```

## Bluetooth

Bluetooth won't work after suspend. To fix this create the script */usr/lib/pm-utils/sleep.d/99bluetooth* to be executed after suspend:

```
 #!/bin/sh
 case "$1" in
  resume)
   rfkill block 2
   rfkill unblock 2
 esac
```
Make it executable with *chmod*


## Other tweaks and fixes

Wifi won't work after restart. Instead do a shutdown and power on

The keyboard does not have all keys. Shortcuts for some missing keys:

<   Shift+AltGr+Z  
\>  Shift+AltGr+X  
Esc Ctrl+AltGr+  


Touchscreen scrolling in Firefox doesn't work. To fix this add to the file `/etc/security/pam_env.conf`

`MOZ_USE_XINPUT2 DEFAULT=1`

In Firefox go to `about:config` and then change

`dow.w3c_touch_events.enabled=1`

To avoid double click when single clicking with the touchpad increase `Double click delay` in Settings Universal access

Increase mouse pointer size Settings Universal access

Install gnome-tweaks and increase font scale in Settings in Fonts

Install dconf and increase launcher icon size in /net/launchpad/plank/docks/dock1/icon-size


## Cameras not working

Cameras (ov2680) don't work. Camera driver ov2680 is available with the kernel, needs to be activated though. The issue is the atomISP driver. See [bug](https://bugzilla.kernel.org/show_bug.cgi?id=109821)