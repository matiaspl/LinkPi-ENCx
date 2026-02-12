# Link Pi ENC1 / TBS 2603SE encoders
Both devices are based on a F3520D mainboard (https://linkpi.cn/archives/870). The TBS however lacks the F3520D_EX2 addon board (so no connectors for HDMI out, analog audio input/output, USB socket, buttons and serial port header - software supports all of those features).

![F3520D with EX2 extension board](https://linkpi.cn/wp-content/uploads/2020/06/ef0b8c93641ae54-1.png)

The base software is shared among whole line of Link Pi encoders, so ENC-Tiny (TinyENC1), ENC1, ENC1V2, ENC2, ENC5, ENC9 and ENCSH (and TBS) have the same ups and downs. 

**UPDATE 8.04.2022**: by the looks of it TinyENC1 seems to actually have a bit different firmware with no telnet/ssh access enabled by default. If anyone's interested in getting inside those boxes consider donating one or sending me the recovery firmware.

**UPDATE 9.11.2023**: The most recent boxes use a new more powerful CPU, the SS5248V100 (from what I read this is a drop-in replacement for Hi35xx chips, namely the Hi3520), more RAM and even larger flash storage.

## Personal disclaimer

I have no connections to the LinkPi company or their developent process. My personal opinion is that they are great devices, more capable than some of 10x as expensive known-brand encoders. It has a well planned and responsive UI. The picture and audio quality is good and everything "just works". I'm kind of used not to expect much from a $100-ish video devices, so getting my hands on and testing ENC1 was an Eureka moment ;)

I tried contacting Link Pi (both dev and sales) about the security issues but unfortunately got no response, that's why I decided to write about all potential and real problems here. 

If LinkPi guys/gals are reading this please check your emails and at least tell me to bugger off ;)

It's a real pity that the Linux system underneath didn't get as much love as the UI, but a cautious linux beginner can make the device more secure in just a few steps.

## Known bugs
* USBCam streaming output freezes if the encoder is left running for longer periods of time, e.g. more than 48h. The snapshot pipeline does not seem affected at the same time.
* Mix pipeline inputs freeze if left running for longer periods of time (e.g. more than 48h). The snapshot and streaming pipelines does not seem affected.

## Undocumented features
### Software
* USBCam allows audio only streaming with a UAC device (e.g. USB microphone)
* internal RTMP server accepts connections from outside using rtmp://enc1_ip/live playpath and any stream key (avoid stream* and sub* keys - they are used internally), so it can be used as a relay
* since _**update_20210927**_ there's a local SRT server (SLS) in listener mode accepting connections with proper streamID. If the encoding party pushes it's stream to srt://enc1_ip:8080?streamid=push/live/XYZ and the player/decoder can connect to the stream here: srt://enc1_ip:8080?streamid=pull/live/XYZ (multiple client connections are supported. 

Note: SLS's built in parser overwrites MPEGTS video headers with H.264 metadata, so you will have hard time playing back HEVC streams sent to SLS. Also, encyption is not supported.
* audio sampling rate up to 96 kHz is supported (but not enabled - it can be added manually by editing the right PHP script)
* ~~OPUS audio codec is supported (software only and disabled by default)~~ (no longer undocumented)
* As of [20231031](https://github.com/matiaspl/LinkPi-ENCx/edit/main/README.md#build-20231031) the encoder has support for adding timestamp information to SEI video stream in-band metadata. UI mentions sinsam ([specs](https://dw.sinsam.com/sinsam/software/%E8%8A%AF%E8%B1%A1SEI%20%E5%B8%A7%E5%90%8C%E6%AD%A5%E6%8A%80%E6%9C%AF%E8%A7%84%E8%8C%83V1.0%20202309.pdf)) and normal (~~I expect this to be "regular" SEI SMPTE-12M timecodes~~ same as sinsam alas not per packet but per GOP - tough luck...)

### Hardware
* the encoders can be powered with 5V using a powerbank or a USB power and a USB-A to DC 5.5x2.5mm barrel connector cable, provided they can deliver ~2 amps. (Carl Mills@EnDeCo)
* there's no power regulator between USB-C and regular USB on ENC1V2, so if you provide 9/12V over USB-C tthe device will boot up, but the same voltage will most likely fry whatever's connected to the other USB ports (Carl Mills@EnDeCo)

## 3rd party apps

Statically compiled armhf applications seem to work fine. I tried ffmpeg and v4l2-ctl and they both worked. 
* LinkPi-oled-logo-converter: https://github.com/YveIce/LinkPi-oled-logo-converter - tool for converting your graphics to OLED boot logo format
* ffmpeg: https://johnvansickle.com/ffmpeg/ 
* v4l2-ctl: https://github.com/9crk/v4l2-ctl - there's a v4l2-ctl.exe file in this repo which is a static armhf binary 
* v4l2ctl-ctl-with-php: https://github.com/wilwad/v4l2-ctl-with-php - allows remote control of USB webcams params (brightness/contrast/backlight, etc.)

## Interesting internal pages (not linked) 

Some of the mentioned pages might no longer be available as the frontend has been significantly cleaned up during UI revamp.
* http://enc1/fac.php - _low level factory settings_ (also uses oled.php, themes.php and remote.php)
* http://enc1/ndireg.php - for entering the NDI license string (sold separately)

Engineering leftovers:

* http://enc1/demo/audio.php - audio mixer, frontend for [AB33EP expansion board](https://gitee.com/LinkPi/Audio/wikis/pages/preview?sort_id=1475161&doc_id=324053)
* http://enc1/demo/audio2.php - backend for audio.php?
* http://enc1/demo/config.html - looks like a "set defaults" script
* http://enc1/demo/rtmp.html
* http://enc1/demo/netex.php
* http://enc1/demo/netex%20-%20拷贝.php (removed in update_20210123)
* http://enc1/demo/timer.html - javascript timer, helpful for latency measurement
* http://enc1/demo/demo.html - face recognition, not working
* http://enc1/wxfunc.php - possibly a part of an old backend, allows non authorised changes
* http://enc1/test.html - possibly a QA leftover, allows unauthorised slideshow view of all inputs
* http://enc1/monitor.php - motion detection
* http://enc1/face.php - face recognition

## Intercom & tally

The intercom system (since version 20211201) is using an unknown UDP-based protocol on port 7000 for communication through a central server (source not public - installer and binaries located here: https://gitee.com/LinkPi/Service/). Judging by the symbols, the server is a QT app built on LinkLib. The client side runs inside the _/link/bin/Encoder_ process and utilizes LinkLib LinkIntercom interfaces 

The tally system is able to utilize vMix and Sinsam (Chinese visual clone of vMix) APIs and the builtin UART (_/dev/ttyAMA1_) as the source of PGM/PVW signals sent downstream. The documentation states, that (as of 8.04.2022) only the vMix integration is complete.

Firmware analysis shows that the system relies on a specific "ttyTally" interface for the tally lights, that presents itself to the LinkPi box as _/dev/ttyUSB0_. Should work with basically any linux supported USB-UART chip (specifically with ESP/ESP32 and Arduino devices). As of 20220712, the udev rules select 10c4:ea60 (Silicon Labs CP210x UART Bridge) as the ttyTally device.

A generic USB UAC soundcard over ALSA seems to be the source and destination as the intercom communication device (may cause trouble if both webcam and intercom were to be used). 

As of version 20220705/20220712 there are udev hotplug rules present for USB audio devices 08dc:0014 (C-Media - Unitek Y-247A) and 12d1:0010 (unknown device with Huawei vendor id - leads me to think it's either a LTE dongle with a headphone jack or a USB-C mobile headphone). Both create a symlink to /dev/headphone which very likely is used as audio IO for the intercom system.

## Default passwords/backdoors
### SSH/telnet

```
root / linkpi.com (Link Pi)
root / turbosig (TBS)
root / linkpi.cn (newer Link Pi boxes)
```

/etc/passwd:
```
(...)
sshd:x:74:74:Privilege-separated SSH:/var/empty/sshd:/sbin/nologin
messagebus:tptRhoI1eT1Ak:1000:1000:Linux User,,,:/home/messagebus:/bin/sh
avahi:uTgaD2s17tOZ.:1002:1002:Linux User,,,:/home/avahi:/bin/sh
netdev:IY9bkphWuof06:1003:1003:Linux User,,,:/home/netdev:/bin/sh
```

**IMPORTANT!** messagebus,avahi and netdev accounts have empty passwords (their hashes can be found here: https://www.seancassidy.me/etc/passwords.txt). This along with enabled _telnetd_ is a _**grave security threat**_! Only sshd account is somewhat protected, because the /etc/shadow that the `:x:` points to doesn't exist and the authentication fails altogether. 

Refer to **Fixes -> Disable telnetd**

### Web
Passwords are stored in `/link/config/passwd.json` as MD5 unsalted chechsums. The defaults are:
```
admin / admin
superadmin / linkpi.com
```
The superadmin account is undocumented, so it's important to either delete it by hand or at least change the password.

### Remote help
There's a built in remote help/E.T. call home feature, that establishes a tunnel connection to Link Pi server using NGROK. If you don't need it, delete the IP address from the ``$remote=`` line in `/link/web/config.php` and reboot.

**UPDATE 05.2021**: the remote management has been made publicly available. If you are a WeChat user you can acquire the bindng code at [[wx.linkpi.cn]] and pair your encoder with the provided web remote access system (open the "Options" -> "Reverse proxy" page of the encoder, paste the correct binding code, turn on the remote access function, and save). 

The system is said to be in early stage of development and occasional downtime may occur.

### ONVIF
A non-protected ONVIF service is running by default with no real way to disable it through the UI

## Fixes & hacks

### Recovering from a bad firmware update 

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

### Add support for other HiLink/Huawei 4G dongles

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

### Create & replace the default NO SIGNAL slate image

By default LinkLib uses `/link/config/nosignal.yuv` image as the placeholder image in case a video source is unavailable while the box is streaming. This file is a 1920x1080 raw semi planar 4:2:0 subsampled YUV bitmap. To create a customized slate, convert your 1920x1080 image with ffmpeg like so:
```
ffmpeg yourfile.png -c:v rawvideo -pix_fmt nv21 nosignal.yuv
```
and overwrite the original. 

### Secure empty password system accounts

To disable the possibility of logging in without a password over telnet, change hashes for users other than root to `:x:` (for example using _vi_). You should end up with the following:

```
(...)
messagebus:x:1000:1000:Linux User,,,:/home/messagebus:/bin/sh
avahi:x:1002:1002:Linux User,,,:/home/avahi:/bin/sh
netdev:x:1003:1003:Linux User,,,:/home/netdev:/bin/sh
```

### Disable telnet daemon (OUTDATED, for recent versions look below)

Comment the _telnetd_ line from `/etc/init.d/rcS`.

### Disable non-secure network services (e.g. telnet, onvif)

Edit /link/config/service.json (thanks to [@Gradinko](https://github.com/Gradinko)) :

 /link/config # more service.json  
 { "telnet":false, "ssh":true, "php":true, "nginx":true, "crond":true, "onvif":true, "ndi":true, "sls":true, "frp":false, "trans":false }

### Set reasonable timezone - OUTDATED
The device uses busybox/libc timezone management, which means you can use `/etc/TZ` for setting the correct time and date. 

Refer to https://www.gnu.org/software/libc/manual/html_node/TZ-Variable.html for creating a line that fits your time zone. 

For CET/CEST (Europe/Warsaw):
```
echo "CET-1CEST,M3.5.0,M10.5.0/3" > /etc/TZ
```

UPDATE: Not needed since 20230322 - time zone can now be configured using the UI.

### Enable key-based SSH login
Most files and folders have UID and GID 1000, some have 0, some have UID 1027 GID 100. I think it's most likely due to an incomplete build script. Due to bad ownership of most of the folders including `/root` subfolders, SSH key based login doesn't work.  

To fix it do:
```
chown root:root ~/
``` 
This will allow _ssh-keygen_ to copy the keys to ~/.ssh.

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

## F3520D pinout

* J1 - USB 0, J4 - USB 1

| 1 | 2 | 3 | 4 |
|---|---|---|---|
| +5V | USB- | USB+ | GND |

* P5 - VGA

| 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 |
|---|---|---|---|---|---|---|---|---|----|
| +5V | NC | NC | VSync | HSync | GND | Red | Green | Blue | GND |

* J16 - UART 0

| 1  | 2  | 3   |
|----|----|-----|
| TX | GND | RX |

## Software/firmware update changelog
Note: The following descriptions are slightly redacted release notes translated from Chinese using Google Translate. 

After the updates up to 20210123 some functions take effect after restarting twice. Newer versions of the firmware seem to perform the restarts automatically.

The devices might come from factory having firmware versions that are not available for download (e.g. 20201111).

### 5.0.0	build 20260130
**Warning: the update broke the Intercom page on my ENC1V1**

* Fixed SLS service memory usage issue
* Added support for TRTC streaming ([Tencent RTC](https://console.cloud.tencent.com/trtc)) to SS524V100 series devices.
* Optimized USB camera audio/video capture and reconnection functions.
* Optimized SRT stream decoding for HI3520DV400, HI3521DV100, and HI3531DV100 series devices.
* Optimized RPC service. A brand-new development mode supports developing device audio/video applications via JavaScript ([LinkJS](https://www.yuque.com/linkpi/linkjs)), supporting SS524, SS528, and SS928 series devices.

Note: This new development mode requires flashing the firmware - it is not included in the upgrade package. Development mode then needs to be enabled through the facory setup (fac.php) URL.

### 4.4.1	build 20251130
This update has the following impact on previous features:
* For versions 20250228 and earlier, updating this firmware will disable the frp service. 
* For systems with versions less than 20250630, directly updating this firmware will overwrite the configuration information on the "Live Streaming" page; please back up your streaming information beforehand. 
* Updating this firmware will disable the infrared RC; please manually access the remote control configuration page and enable it. A second remote control is now supported; you need to select the proper one on the "Infrared Remote Control" page. The remote control function is disabled by default.

Other changes:

* On the [Input] page, the UI for the USB Camera has been adjusted for better consistency with other pages
* Under Extended Functions, a [Serial Control] page has been added, allowing you to operate devices via serial port
* The original "Serial Port, Button" page under Extended Functions has been renamed to [Serial Server] for better alignment with the page's functionality
* The [Carousel] page has been moved to Extended Functions
* Optimized memory usage when using two USB cameras simultaneously on the UVC2 device
* Fixed the issue of increased memory usage due to frequent plugging and unplugging of device input interfaces. 
* Optimized line audio input (active noise reduction) 
* Optimized device stability for receiving SRT streams
* Fixed an issue where AAC audio streams with extensions would not decode correctly
* Fixed an issue where decoding NDI streams using buffered mode would fail
* Fixed an issue with abnormal loading of WeChat mini-programs 
* Added IPv6 support for all models

### 4.3.2 build 20250831
* Fixed an issue where the remote control wouldn't trigger on the ENCS1 model
* Fixed an issue with abnormal SDI output at certain resolutions on the ENC4S model
* Optimized the stability of NTP time synchronization
* Optimized connection stability for the GB28181 platform
* Optimized the preview of H265 encoded video streams on the [Push] page
* Added RIST protocol stream output to the [Streaming] page
* Added a [Carousel] page with support for automatic rotation of single and multi-view layout (this feature is not supported on the MINI model)
* RPC service upgrade

### 4.2.2	build 20250630
This update will overwrite the configuration information on the [Streaming] page. Please back up your streaming information in advance.
* Optimized the stability of Wi-Fi network cards for ENC5V2, ENC2V3, and ENC8
* Fixed an issue where the RTSP stream for the channel could not be obtained via ONVIF when the encoding resolution was set to auto
* Fixed an issue with abnormal audio output on the second HDMI output of the ENC4
* Fixed an issue with low Line Output volume on the MiniEnc1 model
* Added a new [Interface Input] page, including controls for HDMI/SDI/VGA, USB, and Line In. Line In also added noise reduction and gain controls
* Removed the [USB Camera] page from the Lab and integrated its functionality into the [Input] page
* Removed the Video Parameters tab from the [Encode] page and integrated it into the [Input] page and the Rotate/Crop tab on the [Decode] page
* Added support for removing Line In audio from the Mix channel mix
* New interaction logic for [Stream] allows you to specify audio and video sources for each streaming channel

### 4.1.3 build 20250430
* Fixed an issue where the scanned NDI stream and device IP address didn't match
* Removed the invalid network port display on the ENC2V2 model page
* Changed the default EDID for ENC8 and ENCMini models to 1080p
* Fixed an issue where setting a timer for streaming didn't work
* Fixed an issue where remote assistance couldn't be enabled on ENCMini models
* Fixed an issue where SSH login couldn't be enabled on ENCMini models
* New MAC address management mechanism
* Added a custom channel to the RTSP tab on the stream output page to enable Onvif output
* Optimized memory usage for ENCMini models
* Optimized Onvif output
* Optimized WebRTC protocol streaming
* Optimized stability for connecting to the TallyArbiter service
* Optimized device startup speed

### 4.0.0 build 20250228
This update only supports versions 20240131 and later. Do not directly use the upgrade package lower than version 20230426 to downgrade.
* Remove the classic version webpage and related configurations for all models
* Optimize the log feature memory usage
* Optimize the decoding logic of video-only network streams
* Remove the insta360 link from the Laboratory page
* Add a USB camera page to the laboratory, compatible with insta360 control
* Correct the problem of inaccurate acquisition of reserved space for some USB disks when using loop recording
* Correct the problem of discontinuous file names displayed on the [File Management-Loop Recording] tab on the file recording page
* Remove the AUD and BR status display from the OLED display, and add the recording (REC) and push streaming (PUSH) status display

### 3.6.0 build 20241231
This update only supports versions 20240131 and later. Do not directly use the upgrade package lower than version 20230426 to downgrade.
* System log viewing/downloading added to Laboratory page
* Line Output control has been added to the interface output page
* AAC-HE encoding has been added to the encoding settings page for audio encoding
* Loop recording, segmented recording, power-on recording and other functions have been added to the file recording page.
* Corrected the problem that audio mp3 encoding does not work
* Optimized the scaling and performance issues of watermark effects
* Modified some page text descriptions

### 3.5.0 build 20241031
This update only supports versions 20240131 and later. Do not directly use the upgrade package lower than version 20230426 to downgrade
* New OLED screen button interaction
* Fixed the problem that the menu button does not display on small scrrens
* On the live streaming page, fix the problem that there is a probability that the streaming preview cannot be viewed after clicking the save button during streaming
* On the H5 player page, add the function of creating a full-screen playback shortcut for the specified channel
* Optimize the decoding of network streams containing B frames
* Low-level system optimizations

### 3.4.0 build 20240831
* Decoding settings page supports multiple NDI stream decoding output
* Adjust the interactive logic of live streaming page, add WebRTC streaming function
* Video special effects page supports video stream preview (reqires enabling RTMP or WebRTC streams)
* Fixed a rare problem of not being able to scan the onvif stream output by the device
* Optimize the theme function, add the function of setting the main color tone

### 3.3.0 build 20240731
* SRT tab added to the Decoding page
* Added the output of ultra low latency WebRTC preview (when enabled it's available in the H5 player)
* Added one-click stream URL copying in the Streaming output page
* DHCP optimizations

### 3.2.0 build 20240531
* Standard version: decoding settings page adds the function of receiving rtmp streaming, supports authentication streaming
* Fix for ENCSHV2 model for an issue with WIFI being turned on by default when it is first started after flashing
* Optimize frame synchronization function

### 3.1.0 build 20240430
* Standard Edition: Modify the interactive logic of the title search box, and no longer pop up the search box
* Standard Edition: Title search function, add search result keyword highlighting
* Standard Edition: Remove the network input option on the encoding page, and set the network input on the decoding page
* Standard Edition: Correct the problem that there is still a probability of automatically obtaining IP after turning off DHCP in the network settings
* Standard Edition: Fully compatible with preview effects of various resolutions such as vertical screen
* Standard Edition: New remote control interaction logic, the remote control is only supported on some models
* Standard Edition: Correct the problem that some models do not have a 4K option on the encoding setting page
* Standard Edition: Optimize some Http interface logic
* Classic Edition: Correct the problem that there is a probability that the video channel cannot be cut in when switching the layout on the video mixing page
* Custom layout manager, support vertical screen and various custom resolution display

### 3.0.0 build 20240131
* Standard Edition: Fixed some page translation issues
* Standard Edition: Fixed the problem of unsuccessful NDI registration
* Standard Edition: Added title keyword search function
* Added video channel to audio source binding in the layout manager (*I don't see this option in the layout manager, but the Encoder settings Audio tab has a wider set of audio sources available than it had some time ago...*)
* Fixed the problem that NTFS formatted hard disk cannot be automatically mounted
* Optimize some HTTP interface logic
* Optimize file recording function
* Kernel optimization for better UDP stability (ENC1V3, ENCSHV2, ENC4S models only)

### 2.9.0 build 20231229
* Classic version: Fixed the problem that when switching the layout of the specified input source, there is a probability that the input source switch is invalid.
* Standard Edition: Fixed the abnormal page loading problem caused by the integrated communication page not being bound to the broadcasting software.
* Standard Edition: Fixed the problem that after setting LPH, the setting becomes invalid after switching versions.
* Standard version: Added single-click HDMI output function to the decoding setting page
* Remove redundant return values ​​from LPH setting watermark interface
* Network management optimizations

### 2.8.0 build 20231130
* Optimize segment recording function
* NTP synchronization increases timing synchronization interval
* Fixed abnormal recording and streaming issues caused by NTP synchronization
* Corrected the UDP packet reordering problem of ENC1V3, ENCSHV2 and other models
* Fixed the problem that the number of channels with special effects cannot exceed 8
* Following a change to ENCSH models, a new management UI has been added. "Classic" (original) and "standard" (new) versions are available

### 2.7.0 build 20231031
* Added frame synchronization settings ("sinsam/normal/close") to the encoding settings page
* Multi-platform live broadcast page - new push compatibility settings adding the possibility to send H265 streams to YouTube
* Optimize NTP synchronization
* Optimize audio-only network stream encoding logic
* System optimizations

### 2.6.0 build 20230928
* Optimize audio-less network stream encoding logic
* Optimize audioless USB camera encoding logic
* Optimize device NDI encoding frame rate
* Fixed the problem where ENC2V2 would not recognize the connected USB network card
* Fixed the watermark scaling problem caused by switching between low-resolution streams and NOSIGNAL screens.

### 2.5.0 build 20230831
* Fixed the abnormal problem of getting/setting watermarks in the http interface
* Fixed the problem in SRT output where the latency could not be greater than 1000 ms
* Optimize the 4K output quality of ENC1V3 and ENCSHV2 models
* HDMI output adds horizontal mirroring function for ENC1V3, ENCSHV2 and ENC5V2 models
* New Rtty access device method is added in the remote access page under advanced settings
* System optimizations

### 2.4.0 build 20230731
* Fixed the inaccurate display of disk space when mounting a mass storage device
* Fix a small probability of abnormality when restoring the factory settings on ENC5V2 model
* Added Level B standard support to SDI input
* Added remote control support for ENC1V3, ENC4SS, ENCSHV2 models
* Added storage mount settings under extended functions, supports nfs mount, windows shared directory mount, and disk specified partition mount

BUGS: 
* LCD layout config applet no longer works (*tested*)
* chromakey ("green screen") function locks up from time to time (*not tested*)


### 2.3.0 build 20230630
* Fixed the problem of increased memory usage caused by invalid streaming addresses
* Corrected the problem of displaying the playback address on the output settings page
* Added the function of restoring user settings after upgrading through the upgrade package

BUGS: 
* LCD layout config applet no longer works (*tested*)
* chromakey ("green screen") function locks up from time to time (*not tested*)

### 2.2.0 build 20230531
This update only supports version 20230426 and later. For other versions, please download the full flash firmware upgrade to this version. If you need to downgrade, please use the corresponding version flash package. Do not directly use the upgrade package lower than 20230426 to downgrade. **Note that this upgrade will overwrite device configuration information, please back up as needed**

* Added timed enable/disable push function for live streaming on all platforms
* Optimize the Onvif PTZ function of some models
* Optimize integrated communication functions
* Optimize device memory usage
* Adjust the port status display logic on the home page
* Fix roi setting function
* Fixed the problem of inaccurate display of the streaming duration caused by changing the system time after the streaming is enabled
* Corrected the problem that the buttons of the ENC1 model have a small probability of not being triggered
* Support the new Internet-based WeChat applet, no longer rely on the LAN, and truly view it anytime, anywhere

BUGS: 
* LCD layout config applet no longer works
* chromakey ("green screen") function locks up from time to time

### 2.1.0 build 20230426
This update only supports version 20221116 and later. For other versions, please download the full flash firmware upgrade
to this version. If you need to downgrade, please use the corresponding version of the flash package. Do not directly use the upgrade package lower than this version to downgrade. **Note this upgrade Device configuration information will be overwritten, please back it up as needed**

Notice - updating the software from 20230322 to this version softbricked my unit, I had to do a full flash upgrade to get it back.

* Fixed buffer control for B frame decoding
* Reduced CPU usage of MP3 encoding
* Support the latest firmware of insta360Link
* Fixed the abnormality of superimposing special effects of network streams in some cases
* Fixed the abnormal display of decoded illustrations under zooming, rotation, etc.
* Fixed some network stream rotation exceptions in some models
* Solve the issue of ipv6 domain name resolution in case of SRT
* Replace the upgrade package/resource upload component
* Video carousel supports decoding and re-encoding (i.e. U disk playback, streaming)
* Added keying function in the laboratory, that is, green screen keying

### 2.0.0 build 20230322
**This update reportedly changes the way firmware upgrades can be made. The vendor states, that you can only go one version up at a time
**
* Status page - fix for displaying network uplink and downlink status
* Encoding settings - (video parameters) added cropping and rotating the video network stream, added NTSC compatible frame rates for HDMI/SDI channel
* Encoding settings - (audio parameters) the audio source selection supports selecting audio from other video channels
* Output settings - (Play URL settings) added the ability to change the default main stream/sub stream naming scheme
* System settings - added batch configuration export and import, added time zone selection function
* Multiple Push settings - added option to select main stream/sub
* By default the no signal slate is displayed when there is no signal in the network stream - this function can be turned on/off through the fac.php page
* Fixed incorrect translation of some pages and optimized other functions
* A new experimental module is added to the menu bar, including GB28181 support, ROI coding area of interest, Insta360 camera control, Onvif PTZ control, audio and video synchronization adjustment functions (experimental function, use with caution)

### 1.0.0 build 20221116
* Added online firmware search & upgrade function
* rtsp output stream, add account password authentication function
* Add USB disk assistant under extended function
* Add B-frame video stream decoding compatibility
* Fix the problem of obtaining abnormal network status when using a wifi network card
* Optimize the intercom function
* Add support for h265 encoding playback on the h5 player page (playing h265 encoded video streams requires high computer performance, in case of lagging problems, please change to a computer with better performance to play)
* ENC1, ENC1V2, ENCSH models support wifi6 network card version 2.0

### update_20220712
* Fixed a small probability of abnormal audio capture after restarting the device when using a usb camera
* Fixed the problem that the audio of the USB camera could not be captured after the USB camera was plugged and unplugged
* Fixed the problem of the increasing number of opened file handles by the Encoder program caused by an invalid srt stream
* Fix the problem that the custom layout manager is loaded abnormally after the device is bound to wx.linkpi.cn
* All models add HTTP API interface, optimize the URL matching rules of HTTP API
* The login box is centered horizontally and vertically, and the fill color when the password is automatically filled in the input box is removed
* Fix the problem with multi-NIC devices getting abnormal network status
* The intercom function of ENC1, ENC1V2, and ENCSH models is changed to PTT
* Fix the problem of group setting auxiliary stream

### update_20220518
* Fixed the problem that NDI5 cannot decode the H265 stream of NDI4 (NDI|HX2)
* Fixed the problem that the YouTube live broadcast was stuck

*Applicable to any version after 20211201

### update_20220424
* Upgrade to NDI5 (NDI4 users who have purchased authorization need to contact customer service to replace the authorization code)
* Add HTTP-API interface

*Applicable to any version after 20211201

### update_20211201
* Added support for integrated call system and Tally lights [Link to official wiki](https://gitee.com/LinkPi/Encoder/wikis/%E8%BF%9B%E9%98%B6%E6%95%99%E7%A8%8B/%E9%9B%86%E6%88%90%E9%80%9A%E4%BF%A1%E7%B3%BB%E7%BB%9F/%E9%9B%86%E6%88%90%E9%80%9A%E4%BF%A1%E7%B3%BB%E7%BB%9F%E6%A6%82%E8%BF%B0)
* New configuration file import and export function
* New background media service configuration file modification function
* Format verification of parameters such as ip on the network settings page
* Recording support file segmentation
* Support special USB disk or tf card without partition
* Fixed the problem of invalid audio from usb camera
* Fix the abnormal display of OLED recording status
* Fix the problem that the network input will restart when saving parameters

### update_20210927
* Added frp support (https://github.com/fatedier/frp)
* Built-in sls service (https://github.com/Edward-Wu/srt-live-server)
* The streamID attribute is added to the SRT output of the output setting page
* Improve ntp time synchronization
* Optimize audio and video synchronization
* Solve the problem of possible conflict between remote access and srt
* Solve the problem of abnormal increase in cpu occupancy rate under certain circumstances
* Revise the display effect of web theme
* _Low bitrate NDI support (vendor forgot to mention it in the changelog). Low bitate NDI stream follows the sub stream settings._

### update_20210607
* Fix the language switching problem caused by the previous version
* Fix the MP4 recording problem caused by the previous version

*Applicable to any version after 20210123

### update_20210527
* Add remote access function (how to use )
* Add custom layout design interface
* Optimize decoding compatibility
* Optimize decoding fluency (synchronous mode)
* Solve the problem that the network input source cannot be re-encoded after restarting
* Enhanced HDMI input stability

*Applicable to any version after 20210123

### update_20210123
* ENC1V2 adds remote control operation support
* ENCTiny improves stability
* Optimize the file recording operation interface
* Fix css settings conflicting with OEM package in web
* Language selection will be stored on the device, no longer relying solely on cookies
* [3520D] Solve the problem of abnormal HDMI input resolution or abnormal audio in some cases of version 1029
* [3531D] Solve the problem that OLED program will cause high CPU usage
* Solve the problem that the HDMI output resolution may become smaller when the device is repeatedly restarted in version 1029

### update_20201029
* Add ENC1V2 model
* Enhanced HDMI compatibility
* Button function can be turned off
* Fix ENCSH reset problem
* Fix the problem that there is no preview image on the webpage in some cases
* Fixed the problem that there is a certain probability of packet loss when the rtsp large bit rate is

### update_20200823
* The synchronization mode of network input can manually set the buffer time
* Audio gain can be applied to HDMI, SDI, Line and Mix at the same time
* Serial port transparent transmission supports setting of IP, which can realize the application of group pair transparent transmission
* Add video mixing up and down split screen layout, suitable for vertical screen live streaming
* Support turning off video encoding to achieve separate audio output
* Optimize the recording management interface
* Add SRT encryption option
* Support rotation of interlaced video
* Add commonly used USB serial port driver
* Added H5 player page
* Newly added by the immediate DHCP function
* Added network test function
* Optimize the compatibility of sending and receiving with large bit rate
* Optimize the time stamp interval
* Fix youtube audio compatibility
* Fix the problem of converting rtmp to rtmp
* Fix rtmp to carry frame rate information

### update_20200504
* Fixed the issue that NDI authorization will be cleared after the upgrade

### update_20200504
* Added remote assistance function (bottom of system settings)

### update_20200502
* Correct the latency time unit of SRT
* Open 3520D watchdog

### update_20200430
* Added 4G module support
* Fixed the problem of abnormal reconnection when opening multiple SRT channels

### update_20200427
* Fix the wifi management page

### update_20200426
* Modify the SRT user interface
* Added wifi module support
* Fix the exception of Overlay addition, deletion and modification

### update_20200420
* Newly added platform live video preview
* Improve the applet interface

## Links
- https://www.yuque.com/linkpi/encoder/ - Official English documentation
- https://linkpi.gitbook.io/encoder/ - old official English documentation
- https://www.yuque.com/linkpi/encoder/pkfir2g98gwpzymt - upgrade firmware repository
- https://www.yuque.com/linkpi/encoder/kpvg3mbg5ussrx69 - boot ("full flash") firmware repository
- https://www.yuque.com/linkpi/linkjs/ - LinkJS development documentation
- https://gitee.com/LinkPi/ - official Link Pi code repository
- https://gitee.com/LinkPi/Encoder/wikis - official docs, good source of knowledge about the internals and firmware
- https://linkpi.cn/archives/1388 - HTTP API ([Google translate](https://linkpi-cn.translate.goog/archives/1388?_x_tr_sl=auto&_x_tr_tl=en&_x_tr_hl=pl&_x_tr_pto=wapp))
- https://gitee.com/LinkPi/Encoder/issues - bug list (I'm not sure any developer ever looked at those)
- https://blog.csdn.net/weixin_41486034 - Chinese blog with a lot of fun ENCx use cases and ideas (most likely ran by a Link Pi developer or a tech-savvy salesperson)
- https://linkpi.cn/archives/444 - a repost of a CSDN article on extensive ENC5 testing
- http://wiki.endeco.xyz/ - english manual for rebranded and secured LinkPi encoders (no longer operational ;( )
  
