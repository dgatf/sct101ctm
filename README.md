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

7.Webcams wont work

8.SD card error
