# Link Pi ENC1 / TBS 2603SE encoders
Both devices are based on a F3520D mainboard (https://linkpi.cn/archives/870). The TBS however lacks the F3520D_EX2 addon board (so no connectors for HDMI out, analog audio input/output, USB socket, buttons and serial port header, software however supports all of those features).

![F3520D with EX2 extension board](https://linkpi.cn/wp-content/uploads/2020/06/ef0b8c93641ae54-1.png)

The base software is shared among whole line of Link Pi encoders, so ENC-Tiny (TinyENC1), ENC1, ENC1V2, ENC2, ENC5, ENC9 and ENCSH (and TBS) have the same ups and downs. 

**UPDATE 8.04.2022**: by the looks of it TinyENC1 seems to actually have a bit different firmware with no telnet/ssh access enabled by default. If anyone's interested in getting inside those boxes consider donating one or sending me the recovery firmware.

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
* internal RTMP server accepts connections from outside using rtmp://enc1_ip/live playpath and any stream key (avoid stream* and sub* keys - they are used internally)
* since _**update_20210927**_ there's a local SRT server (SLS) in listener mode accepting connections with proper streamID. If the encoding party pushes it's stream to srt://enc1_ip:8080?streamid=push/live/XYZ and the player/decoder can connect to the stream here: srt://enc1_ip:8080?streamid=pull/live/XYZ (multiple client connections are supported. Note: SLS built in parser overwrites MPEGTS video headers with H.264 metadata, so you will have hard time playing back HEVC streams sent to SLS. Also, encyption is not supported.
* audio sampling rate up to 96 kHz is supported (but not enabled - it can be added manually by editing the right PHP script)
* OPUS audio codec is supported (software only and disabled by default)
* Encoder has support for adding timestamp information to SEI video stream in-band metadata. UI mentions sinsam ([specs](https://dw.sinsam.com/sinsam/software/%E8%8A%AF%E8%B1%A1SEI%20%E5%B8%A7%E5%90%8C%E6%AD%A5%E6%8A%80%E6%9C%AF%E8%A7%84%E8%8C%83V1.0%20202309.pdf)) and normal (I expect this to be "regular" SEI SMPTE-12M timecodes)

### Hardware
* the encoders can be powered with 5V using a powerbank or a USB power and a USB-A to DC 5.5x2.5mm barrel connector cable, provided they can deliver ~2 amps. (Carl Mills@EnDeCo)
* there's no power regulator between USB-C and regular USB on ENC1V2, so if you provide 9/12V over USB-C tthe device will boot up, but the same voltage will most likely fry whatever's connected to the other USB ports (Carl Mills@EnDeCo)

## 3rd party apps

Statically compiled armhf applications seem to work fine. I tried ffmpeg and v4l2-ctl and they both worked. 

* ffmpeg: https://johnvansickle.com/ffmpeg/ 
* v4l2-ctl: https://github.com/9crk/v4l2-ctl - there's a v4l2-ctl.exe file in this repo which is a static armhf binary 
* v4l2ctl-ctl-with-php: https://github.com/wilwad/v4l2-ctl-with-php - allows remote control of USB webcams params (brightness/contrast/backlight, etc.)

## Interesting internal pages (not linked/unused/model-specific) 

* http://enc1/test.html - possibly a QA leftover, allows unauthorised slideshow view of all inputs
* http://enc1/fac.php - _low level factory settings_ (also uses oled.php, themes.php and remote.php)
* http://enc1/ndireg.php - for entering the NDI license string (sold separately)
* http://enc1/remote.php and http://enc1/remotelp.php - IR remote configuration (supported on ENC1V2)
* http://enc1/monitor.php - motion detection
* http://enc1/face.php - face recognition

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

## Intercom & tally

The intercom system (since version 20211201) is using an unknown UDP-based protocol on port 7000 for communication through a central server (source not public - installer and binaries located here: https://gitee.com/LinkPi/Service/). Judging by the symbols, the server is a QT app built on LinkLib. The client side runs inside the _/link/bin/Encoder_ process and utulizes LinkLib LinkIntercom interfaces 

The tally system is able to utilize vMix and Sinsam (Chinese visual clone of vMix) APIs and the builtin UART (_/dev/ttyAMA1_) as the source of PGM/PVW signals sent downstream. The documentation states, that (as of 8.04.2022) only the vMix integration is complete.

Firmware analysis shows that the system relies on a specific "ttyTally" interface for the tally lights, that presents itself to the LinkPi box as _/dev/ttyUSB0_. Should work with basically any linux supported USB-UART chip (specifically with ESP/ESP32 and Arduino devices). As of 20220712, the udev rules select 10c4:ea60 (Silicon Labs CP210x UART Bridge) as the ttyTally device.

A generic USB UAC soundcard over ALSA seems to be the source and destination as the intercom communication device (may cause trouble if both webcam and intercom were to be used). 

As of version 20220705/20220712 there are udev hotplug rules present for USB audio devices 08dc:0014 (C-Media - Unitek Y-247A) and 12d1:0010 (unknown device with Huawei vendor id - leads me to think it's either a LTE dongle with a headphone jack or a USB-C mobile headphone). Both create a symlink to /dev/headphone which very likely is used as audio IO for the intercom system.

## Default passwords/backdoors
### SSH/telnet

```
root / linkpi.com (Link Pi)
root / turbosig (TBS)
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

### Disable telnet daemon

Comment the _telnetd_ line from `/etc/init.d/rcS`.

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

## Software/firmware update changlelog
After the updates up to 20210123 some functions take effect after restarting twice. Newer versions of the firmware seem to perform the restarts automatically.

The devices come from factory having versions of firmware not available for download (e.g. 20201111)

### build 20231031
* Added frame synchronization settings ("sinsam/normal/close") to the encoding settings page
* Multi-platform live broadcast page - new push compatibility settings adding the possibility to send H265 streams to YouTube
* Optimize NTP synchronization
* Optimize audio-only network stream encoding logic
* System optimizations

### build 20230928
* Optimize audio-less network stream encoding logic
* Optimize audioless USB camera encoding logic
* Optimize device NDI encoding frame rate
* Fixed the problem where ENC2V2 would not recognize the connected USB network card
* Fixed the watermark scaling problem caused by switching between low-resolution streams and NOSIGNAL screens.

### build 20230831
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
This update only supports version 20230426 and later. For other versions, please download the full flash firmware upgrade to
this version. If you need to downgrade, please use the corresponding version flash package. Do not directly use the upgrade package lower than 20230426 to downgrade. **Note that this upgrade will Overwrite device configuration information, please back up as needed**

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
- https://linkpi.gitbook.io/encoder/ - Official English documentation
- https://gitee.com/LinkPi/ - official Link Pi code repository
- https://gitee.com/LinkPi/Encoder/wikis - official docs, good source of knowledge about the internals and firmware
- https://linkpi.cn/archives/1388 - HTTP API ([Google translate](https://linkpi-cn.translate.goog/archives/1388?_x_tr_sl=auto&_x_tr_tl=en&_x_tr_hl=pl&_x_tr_pto=wapp))
- https://gitee.com/LinkPi/Encoder/issues - bug list (I'm not sure any developer ever looked at those)
- https://blog.csdn.net/weixin_41486034 - Chinese blog with a lot of fun ENCx use cases and ideas (most likely ran by a Link Pi developer or a tech-savvy salesperson)
- https://linkpi.cn/archives/444 - a repost of a CSDN article on extensive ENC5 testing
- http://wiki.endeco.xyz/ - english manual for rebranded and secured LinkPi encoders (no longer operational ;( )

