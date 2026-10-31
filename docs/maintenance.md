# Firmware recovery and device fixes

## Recovering from a bad firmware update
Full flash packages differ from the upgrade packages. Upgrades are basically .tar files that - generally speaking - include the files that changed since the last firmware version, and full flash are partition images. You need to grab the one for your device from the following link:
 https://gitee.com/LinkPi/Encoder/wikis/%E5%8D%87%E7%BA%A7&%E5%88%B7%E6%9C%BA/%E5%88%B7%E6%9C%BA%E5%8C%85
and follow the instructions below:

1. Prepare a USB pendrive, format it as FAT32, single partition (with no hidden partitions) (I recommend using one with an activity LED - makes time spent in points 5-6 more bearable)
2. Unzip all the flashing packages of the corresponding model to the root directory of the USB disk
3. When the encoder is powered off, insert the USB disk into the USB port of the encoder
4. Press and hold the 'DEF' button of the encoder (with a toothpick) and turn on the encoder power
5. Release the DEF button after you see 'UPDATING' on the OLED screen or 'System Updating' on the HDMI screen
6. Wait for the firmware flashing to complete (the logo appears on the OLED screen or the logo appears on the HDMI screen)
7. After the system is flashed, the default IP will be restored (192.168.1.217) which corresponds to the flashing package of multiple models. You need to manually visit http://192.168.1.217/fac.php - after logging in select and confirm appropriate model. Restart after you're done.

If you have problems navigating LinkPi gitee repositories Chrome's built-in translation engine does a very good job.

## Add support for other HiLink/Huawei 4G dongles
Check the VID and PID of your modem using `lsusb`, and alter accordingly:

/etc/udev/rules.d/11-usb-hotplug.rules
```
(...)
ATTRS{idVendor}=="12d1", ATTRS{idProduct}=="1f01",  RUN+="/etc/udev/usb4g.sh"
SUBSYSTEM=="net", ATTRS{idVendor}=="12d1", ATTRS{idProduct}=="14db", KERNEL=="eth*",  RUN+="/etc/udev/usbUp.sh %k"
(...)
```
/etc/udev/usb4g.sh
```
#!/bin/sh
/usr/bin/usb_modeswitch -v 0x12d1 -p 0x1f01 -J
```
(the usbUp.sh script just runs udhcpc on the right interface, no need to change anything there)

## Create & replace the default NO SIGNAL slate image
By default LinkLib uses `/link/config/nosignal.yuv` image as the placeholder image in case a video source is unavailable while the box is streaming. This file is a 1920x1080 raw semi planar 4:2:0 subsampled YUV bitmap. To create a customized slate, convert your 1920x1080 image with ffmpeg like so:
```
ffmpeg yourfile.png -c:v rawvideo -pix_fmt nv21 nosignal.yuv
```
and overwrite the original.

## Set reasonable timezone - OUTDATED
The device uses busybox/libc timezone management, which means you can use `/etc/TZ` for setting the correct time and date.

Refer to https://www.gnu.org/software/libc/manual/html_node/TZ-Variable.html for creating a line that fits your time zone.

For CET/CEST (Europe/Warsaw):
```
echo "CET-1CEST,M3.5.0,M10.5.0/3" > /etc/TZ
```

UPDATE: Not needed since 20230322 - time zone can now be configured using the UI.

## Enable key-based SSH login
Most files and folders have UID and GID 1000, some have 0, some have UID 1027 GID 100. I think it's most likely due to an incomplete build script. Due to bad ownership of most of the folders including `/root` subfolders, SSH key based login doesn't work.

To fix it do:
```
chown root:root ~/
```
This will allow _ssh-keygen_ to copy the keys to ~/.ssh.
