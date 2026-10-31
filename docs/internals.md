# Firmware internals and observations

## Other findings and comments
1. The whole UI is written in PHP
2. The software is common for devices with and without OLED display - specific functions are enabled through http://enc1/fac.php - this way you can enable USB recording, UVC webcam input (and USB UAC audio devices in that matter), NDI support, video player, UART support and the assignment of GPIO buttons to specific functions.
If enabling doesn't work, you may need to look into `/link/web/config.php` and add the parameters to the appropriate files manually (e.g. $NDI=true and $OPUS=true). Use `/link/fac/DEF/web/config.php` for reference.
3. The device does hardly any internal logging - there are only nginx-rtmp fork (supporting rtmp and rtmp/flv over http) and php-fpm logs. To enable system logging (until next restart) do:
```
mkdir /var/log
syslogd
```
or enable it in the startup scripts.

4. The device has 256 MB of internal memory. Quite unusual for this type of device, the whole filesystem is mounted in writable mode:
```
~ # df -h
Filesystem                Size      Used Available Use% Mounted on
ubi0:ubifs              214.7M     62.3M    152.4M  29% /
tmpfs                    95.7M         0     95.7M   0% /dev
tmpfs                    95.7M    248.0K     95.4M   0% /tmp
```
```
~ # mount
rootfs on / type rootfs (rw)
ubi0:ubifs on / type ubifs (rw,sync,relatime)
proc on /proc type proc (rw,nosuid,nodev,relatime)
sysfs on /sys type sysfs (rw,nosuid,nodev,relatime)
tmpfs on /dev type tmpfs (rw,relatime)
tmpfs on /tmp type tmpfs (rw,relatime)
devpts on /dev/pts type devpts (rw,relatime,mode=600)
```

6. The following drivers are included in the kernel:
- cdc_ncm (4G modems)
- uvcvideo (UVC devices)
- ALSA (audio including UAC devices)
...and not much more.

There are however vendor provided additional modules:
- wifi - 88x2cu i 8821cu
- Gennum SDI deserializer - gs2971a
- six channel Texas Instruments ADC audio codec - tlv320aic31
- hardware SHA256 accelerator - atsha204a
- unknown GPIO multiplexer(s)

## Getting information from HiSilicon HW accelerated AV backend
(https://www.programmersought.com/article/52274083303

* VPSS (Video Processing Subsystem) `cat /proc/umap/vpss`
* VI (Video Input) `cat /proc/umap/vi`
* VO (Video Output) `cat /proc/umap/vo`
* VEnc (Video Encoder) `cat /proc/umap/venc`
* VDec (Video Decoder) `cat /proc/umap/vdec`
* AI (Audio Input) `cat /proc/umap/ai`
* SYS (System) `cat /proc/umap/sys`

### Sink/source-specific parameters
```
cat /proc/umap/hdmi0
cat /proc/umap/hdmi0_sink
cat /proc/umap/hdmi0_vo
```

### Setting MPP log level and getting the contents
```
cat /dev/logmpp
echo venc=7 > /proc/umap/logmpp
cat /dev/logmpp
echo venc=3 > /proc/umap/logmpp
```

Getting information about memory used by the backend:
```
cat /proc/media-mem
```
