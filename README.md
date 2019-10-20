#Install Ubuntu 19.04 on Schneider SCT101CTM

This document describes the process to install Ubuntu 19.04 on Schneider SCT101CTM. It is required a working installation of Ubuntu

## Prepare the ISO in a USB stick on a Ubuntu installation

Download [ubuntu-19.10-desktop-amd64.iso](https://ubuntu.com/download/desktop/thank-you?country=ES&version=19.10&architecture=amd64)

Install isorespin. This tablet requires a 32bit UEFI boot to load a 64bits OS. Ubuntu comes with 64bits bootloader. To replace with a 32bit UEFI bootloader we use install [isorespin](http://linuxiumcomau.blogspot.com/2017/06/customizing-ubuntu-isos-documentation.html) script

Create linuxium-ubuntu-19.04-desktop-amd64.iso:

`isorespin.sh -i ubuntu-19.04-desktop-amd64.iso`

Copy the iso to USB stick with Startup Disk Creator

## Change UEFI boot order in Windows on Schneider

Boot Windows 10 and insert USB stick with the ISO 

Open a terminal window with cmd and reboot into UEFI:

`shutdown /r /fw`

(In Ubuntu: `systemctl reboot --firmware-setup`)

Go to Boot tab and change the boot order or override boot to the USB, then Save & exit

## Install Ubuntu

Install ubuntu from grub (or try before installing). It is recommended to delete the entire SSD as it is only 32GB, barely enough for dual boot, altohugh is doable (not covered here)

## Touchscreen driver

Wifi, bluetooth and audio works out of the box. For the touchscreen it is needed to install the driver and firmware. Thanks to [gsl-firmware](https://github.com/onitake/gsl-firmware) I've ported the firmware to linux. Use files in [touchscreen](touchscreen) folder. There are two open source drivers: gslx680_ts_acpi and silead_ts

### gslx680_ts_acpi (the easy way)

Copy [silead_ts.fw](touchscreen/silead_ts.fw) to /lib/firmware

10.Compile the driver [gslx680_ts_acpi](https://github.com/onitake/gslx680-acpi) to obtain gslx680_ts_acpi.ko. You can test it with `insmod gslx680_ts_acpi.ko`. You may need to recompile for different kernels

To do this permanent:

`sudo cp gslx680_ts_acpi.ko /lib/modules/$(uname -r)/

sudo nano /etc/modules-load.d/gslx680_ts_acpi.conf
 gslx680_ts_acpi

sudo depmod`

To test:
`sudo modprobe gslx680_ts_acpi)
lsmod | grep gslx680_ts_acpi`

### Kernel driver silead_ts (the hard way recommended)

Todo

## Fix screen rotation

To fix wrong screen orientation create a udev rule for the sensor in the file /etc/udev/hwdb.d/61-sensor-local.hwdb

`sensor:modalias:acpi:BOSC0200*:dmi:bvnAmericanMegatrendsInc.:bvrSCH12i.WJ210Z.KtBJRCA03*

 ACCEL_MOUNT_MATRIX=-1, 0, 0; 0, 1, 0; 0, 0, 1`

Update udev rules:

`sudo systemd-hwdb update
sudo udevadm trigger -v -p DEVNAME=/dev/iio:device0`

Shutdown and power on


## Bluetooth

Bluetooth won't work after suspend. To fix this it is needed to run a script after suspend:

'sudo nano /usr/lib/pm-utils/sleep.d/99bluetooth

 #!/bin/sh  
 case "$1" in  
  resume)  
   rfkill block 2  
   rfkill unblock 2  
 esac'

## Other tweaks and fixes

Wifi won't work after restart. Instead do a shutdown and power on

The keyboard does not have all keys. Shortcuts for some missing keys:

<   Shift+AltGr+Z  
\>  Shift+AltGr+X  
Esc Ctrl+AltGr+  


Touchscreen scrolling in Firefox doesn't work. To fix this add to /etc/security/pam_env.conf:

`MOZ_USE_XINPUT2 DEFAULT=1`

In Firefox go to about:config and then change:

dow.w3c_touch_events.enabled=1

Webcams won't work. See [bug](https://bugzilla.kernel.org/show_bug.cgi?id=109821)

SD card error. It was working with Ubuntu 16. No solution yet