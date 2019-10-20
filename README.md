Install Ubuntu 19.04 on Schneider SCT101CTM

1.Install isorespin and execute:
isorespin.sh -i ubuntu-19.04-desktop-amd64.iso
This will create linuxium-ubuntu-19.04-desktop-amd64.iso

2.Copy the iso with USB creator

3.Touchscreen. Use drivers in touchscreen folder
gslx680_ts_acpi
Compile gslx680_ts_acpi.ko
Copy silead_ts.fw to /lib/firmware
Then insmod gslx680_ts_acpi.ko

kernel

4.Bluetooth after suspend:
rfkill block 2
rfkill unblock 2

5.Wifi wont work after restart. Shutdown and power on

6.Screen rotation

7.Touchscreen Scrolling firefox
/etc/security/pam_env.conf
MOZ_USE_XINPUT2 DEFAULT=1
Firefox about:config
dow.w3c_touch_events.enabled=1

8.Webcams wont work

9.SD card error

10.Keyboard. Missing keys
< Shift+AltGr+Z
> Shift+AltGr+X
Esc Ctrl+AltGr+`

