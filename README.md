Install Ubuntu 19.04 on Schneider SCT101CTM

1.Install isorespin and execute:
isorespin.sh -i ubuntu-19.04-desktop-amd64.iso

2.Copy the iso using USB creator

3.Touchscreen. Use drivers in touchscreen folder

4.Bluetooth after suspend:
rfkill block 2
rfkill unblock 2

5.Wifi wont work after restart. Shutdown and power on
