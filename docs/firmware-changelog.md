# Software and firmware update changelog
Note: The following descriptions are slightly redacted release notes translated from Chinese using Google Translate.

After the updates up to 20210123 some functions take effect after restarting twice. Newer versions of the firmware seem to perform the restarts automatically.

The devices might come from factory having firmware versions that are not available for download (e.g. 20201111).

## 5.2.0 build 20260531
* Optimized USB camera capture and reconnection functions
* Optimized the Line Input capture and Line Output feature for MINI models
* Optimized Onvif service memory usage
* Optimized the interaction of the [Custom Layout Manager]
* Fixed an issue where restarting the device via the [System Settings] page might result in a second restart if the page wasn't refreshed
* Fixed an issue where the file carousel function on the [Decoding Settings] page might fail to read files occasionally
* Removed the push streaming option from the [Streaming Output] page; please visit the 【Live Streaming】 page to push streams.
* Added the ability to add specified video files to the carousel on the [Auto Carousel] page under Extended Functions
* Added the ability to add filename annotations to video files on the 'File Management - Regular Recording' tab of the [File Recording] page under Extended Functions
* Added a [Storage Management] page under Experimental Functions, allowing management of files on mounted devices, including deleting/moving/uploading files, etc.
* Added an image carousel function to the [Decoding Settings] page, allowing the use of image channels as the background of Mix channels.
Note: Due to the addition of a new image carousel video channel, the default playback address of the Mix channel will change.

## 5.1.0	build 20260331
* Added USB microphone control.
* Optimized 4K HDMI signal input.
* Added ES6 syntax support to the JS Script engine.
* JS Encoder ([LinkJS](https://www.yuque.com/linkpi/linkjs)) development is complete; welcome to switch to development mode for testing (currently in beta).

Note: The new development mode is not included in the upgrade package (full firmware reflash is required). Development mode then needs to be enabled through the facory setup (fac.php) URL.

## 5.0.0	build 20260130
**Warning: the update broke the Intercom page on my ENC1V1** (restoring default settings can very likely fix it)

* Fixed SLS service memory usage issue.
* Added support for TRTC streaming ([Tencent RTC](https://console.cloud.tencent.com/trtc)) to SS524V100 series devices.
* Optimized USB camera audio/video capture and reconnection functions.
* Optimized SRT stream decoding for HI3520DV400, HI3521DV100, and HI3531DV100 series devices.
* Optimized RPC service. A brand-new development mode supports developing device audio/video applications via JavaScript ([LinkJS](https://www.yuque.com/linkpi/linkjs)), supporting SS524, SS528, and SS928 series devices.

Note: The new development mode is not included in the upgrade package (full firmware reflash is required). Development mode then needs to be enabled through the facory setup (fac.php) URL.

## 4.4.1	build 20251130
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

## 4.3.2 build 20250831
* Fixed an issue where the remote control wouldn't trigger on the ENCS1 model
* Fixed an issue with abnormal SDI output at certain resolutions on the ENC4S model
* Optimized the stability of NTP time synchronization
* Optimized connection stability for the GB28181 platform
* Optimized the preview of H265 encoded video streams on the [Push] page
* Added RIST protocol stream output to the [Streaming] page
* Added a [Carousel] page with support for automatic rotation of single and multi-view layout (this feature is not supported on the MINI model)
* RPC service upgrade

## 4.2.2	build 20250630
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

## 4.1.3 build 20250430
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

## 4.0.0 build 20250228
This update only supports versions 20240131 and later. Do not directly use the upgrade package lower than version 20230426 to downgrade.
* Remove the classic version webpage and related configurations for all models
* Optimize the log feature memory usage
* Optimize the decoding logic of video-only network streams
* Remove the insta360 link from the Laboratory page
* Add a USB camera page to the laboratory, compatible with insta360 control
* Correct the problem of inaccurate acquisition of reserved space for some USB disks when using loop recording
* Correct the problem of discontinuous file names displayed on the [File Management-Loop Recording] tab on the file recording page
* Remove the AUD and BR status display from the OLED display, and add the recording (REC) and push streaming (PUSH) status display

## 3.6.0 build 20241231
This update only supports versions 20240131 and later. Do not directly use the upgrade package lower than version 20230426 to downgrade.
* System log viewing/downloading added to Laboratory page
* Line Output control has been added to the interface output page
* AAC-HE encoding has been added to the encoding settings page for audio encoding
* Loop recording, segmented recording, power-on recording and other functions have been added to the file recording page.
* Corrected the problem that audio mp3 encoding does not work
* Optimized the scaling and performance issues of watermark effects
* Modified some page text descriptions

## 3.5.0 build 20241031
This update only supports versions 20240131 and later. Do not directly use the upgrade package lower than version 20230426 to downgrade
* New OLED screen button interaction
* Fixed the problem that the menu button does not display on small scrrens
* On the live streaming page, fix the problem that there is a probability that the streaming preview cannot be viewed after clicking the save button during streaming
* On the H5 player page, add the function of creating a full-screen playback shortcut for the specified channel
* Optimize the decoding of network streams containing B frames
* Low-level system optimizations

## 3.4.0 build 20240831
* Decoding settings page supports multiple NDI stream decoding output
* Adjust the interactive logic of live streaming page, add WebRTC streaming function
* Video special effects page supports video stream preview (reqires enabling RTMP or WebRTC streams)
* Fixed a rare problem of not being able to scan the onvif stream output by the device
* Optimize the theme function, add the function of setting the main color tone

## 3.3.0 build 20240731
* SRT tab added to the Decoding page
* Added the output of ultra low latency WebRTC preview (when enabled it's available in the H5 player)
* Added one-click stream URL copying in the Streaming output page
* DHCP optimizations

## 3.2.0 build 20240531
* Standard version: decoding settings page adds the function of receiving rtmp streaming, supports authentication streaming
* Fix for ENCSHV2 model for an issue with WIFI being turned on by default when it is first started after flashing
* Optimize frame synchronization function

## 3.1.0 build 20240430
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

## 3.0.0 build 20240131
* Standard Edition: Fixed some page translation issues
* Standard Edition: Fixed the problem of unsuccessful NDI registration
* Standard Edition: Added title keyword search function
* Added video channel to audio source binding in the layout manager (*I don't see this option in the layout manager, but the Encoder settings Audio tab has a wider set of audio sources available than it had some time ago...*)
* Fixed the problem that NTFS formatted hard disk cannot be automatically mounted
* Optimize some HTTP interface logic
* Optimize file recording function
* Kernel optimization for better UDP stability (ENC1V3, ENCSHV2, ENC4S models only)

## 2.9.0 build 20231229
* Classic version: Fixed the problem that when switching the layout of the specified input source, there is a probability that the input source switch is invalid.
* Standard Edition: Fixed the abnormal page loading problem caused by the integrated communication page not being bound to the broadcasting software.
* Standard Edition: Fixed the problem that after setting LPH, the setting becomes invalid after switching versions.
* Standard version: Added single-click HDMI output function to the decoding setting page
* Remove redundant return values ​​from LPH setting watermark interface
* Network management optimizations

## 2.8.0 build 20231130
* Optimize segment recording function
* NTP synchronization increases timing synchronization interval
* Fixed abnormal recording and streaming issues caused by NTP synchronization
* Corrected the UDP packet reordering problem of ENC1V3, ENCSHV2 and other models
* Fixed the problem that the number of channels with special effects cannot exceed 8
* Following a change to ENCSH models, a new management UI has been added. "Classic" (original) and "standard" (new) versions are available

## 2.7.0 build 20231031
* Added frame synchronization settings ("sinsam/normal/close") to the encoding settings page
* Multi-platform live broadcast page - new push compatibility settings adding the possibility to send H265 streams to YouTube
* Optimize NTP synchronization
* Optimize audio-only network stream encoding logic
* System optimizations

## 2.6.0 build 20230928
* Optimize audio-less network stream encoding logic
* Optimize audioless USB camera encoding logic
* Optimize device NDI encoding frame rate
* Fixed the problem where ENC2V2 would not recognize the connected USB network card
* Fixed the watermark scaling problem caused by switching between low-resolution streams and NOSIGNAL screens.

## 2.5.0 build 20230831
* Fixed the abnormal problem of getting/setting watermarks in the http interface
* Fixed the problem in SRT output where the latency could not be greater than 1000 ms
* Optimize the 4K output quality of ENC1V3 and ENCSHV2 models
* HDMI output adds horizontal mirroring function for ENC1V3, ENCSHV2 and ENC5V2 models
* New Rtty access device method is added in the remote access page under advanced settings
* System optimizations

## 2.4.0 build 20230731
* Fixed the inaccurate display of disk space when mounting a mass storage device
* Fix a small probability of abnormality when restoring the factory settings on ENC5V2 model
* Added Level B standard support to SDI input
* Added remote control support for ENC1V3, ENC4SS, ENCSHV2 models
* Added storage mount settings under extended functions, supports nfs mount, windows shared directory mount, and disk specified partition mount

BUGS:
* LCD layout config applet no longer works (*tested*)
* chromakey ("green screen") function locks up from time to time (*not tested*)


## 2.3.0 build 20230630
* Fixed the problem of increased memory usage caused by invalid streaming addresses
* Corrected the problem of displaying the playback address on the output settings page
* Added the function of restoring user settings after upgrading through the upgrade package

BUGS:
* LCD layout config applet no longer works (*tested*)
* chromakey ("green screen") function locks up from time to time (*not tested*)

## 2.2.0 build 20230531
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

## 2.1.0 build 20230426
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

## 2.0.0 build 20230322
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

## 1.0.0 build 20221116
* Added online firmware search & upgrade function
* rtsp output stream, add account password authentication function
* Add USB disk assistant under extended function
* Add B-frame video stream decoding compatibility
* Fix the problem of obtaining abnormal network status when using a wifi network card
* Optimize the intercom function
* Add support for h265 encoding playback on the h5 player page (playing h265 encoded video streams requires high computer performance, in case of lagging problems, please change to a computer with better performance to play)
* ENC1, ENC1V2, ENCSH models support wifi6 network card version 2.0

## update_20220712
* Fixed a small probability of abnormal audio capture after restarting the device when using a usb camera
* Fixed the problem that the audio of the USB camera could not be captured after the USB camera was plugged and unplugged
* Fixed the problem of the increasing number of opened file handles by the Encoder program caused by an invalid srt stream
* Fix the problem that the custom layout manager is loaded abnormally after the device is bound to wx.linkpi.cn
* All models add HTTP API interface, optimize the URL matching rules of HTTP API
* The login box is centered horizontally and vertically, and the fill color when the password is automatically filled in the input box is removed
* Fix the problem with multi-NIC devices getting abnormal network status
* The intercom function of ENC1, ENC1V2, and ENCSH models is changed to PTT
* Fix the problem of group setting auxiliary stream

## update_20220518
* Fixed the problem that NDI5 cannot decode the H265 stream of NDI4 (NDI|HX2)
* Fixed the problem that the YouTube live broadcast was stuck

*Applicable to any version after 20211201

## update_20220424
* Upgrade to NDI5 (NDI4 users who have purchased authorization need to contact customer service to replace the authorization code)
* Add HTTP-API interface

*Applicable to any version after 20211201

## update_20211201
* Added support for integrated call system and Tally lights [Link to official wiki](https://gitee.com/LinkPi/Encoder/wikis/%E8%BF%9B%E9%98%B6%E6%95%99%E7%A8%8B/%E9%9B%86%E6%88%90%E9%80%9A%E4%BF%A1%E7%B3%BB%E7%BB%9F/%E9%9B%86%E6%88%90%E9%80%9A%E4%BF%A1%E7%B3%BB%E7%BB%9F%E6%A6%82%E8%BF%B0)
* New configuration file import and export function
* New background media service configuration file modification function
* Format verification of parameters such as ip on the network settings page
* Recording support file segmentation
* Support special USB disk or tf card without partition
* Fixed the problem of invalid audio from usb camera
* Fix the abnormal display of OLED recording status
* Fix the problem that the network input will restart when saving parameters

## update_20210927
* Added frp support (https://github.com/fatedier/frp)
* Built-in sls service (https://github.com/Edward-Wu/srt-live-server)
* The streamID attribute is added to the SRT output of the output setting page
* Improve ntp time synchronization
* Optimize audio and video synchronization
* Solve the problem of possible conflict between remote access and srt
* Solve the problem of abnormal increase in cpu occupancy rate under certain circumstances
* Revise the display effect of web theme
* _Low bitrate NDI support (vendor forgot to mention it in the changelog). Low bitate NDI stream follows the sub stream settings._

## update_20210607
* Fix the language switching problem caused by the previous version
* Fix the MP4 recording problem caused by the previous version

*Applicable to any version after 20210123

## update_20210527
* Add remote access function (how to use )
* Add custom layout design interface
* Optimize decoding compatibility
* Optimize decoding fluency (synchronous mode)
* Solve the problem that the network input source cannot be re-encoded after restarting
* Enhanced HDMI input stability

*Applicable to any version after 20210123

## update_20210123
* ENC1V2 adds remote control operation support
* ENCTiny improves stability
* Optimize the file recording operation interface
* Fix css settings conflicting with OEM package in web
* Language selection will be stored on the device, no longer relying solely on cookies
* [3520D] Solve the problem of abnormal HDMI input resolution or abnormal audio in some cases of version 1029
* [3531D] Solve the problem that OLED program will cause high CPU usage
* Solve the problem that the HDMI output resolution may become smaller when the device is repeatedly restarted in version 1029

## update_20201029
* Add ENC1V2 model
* Enhanced HDMI compatibility
* Button function can be turned off
* Fix ENCSH reset problem
* Fix the problem that there is no preview image on the webpage in some cases
* Fixed the problem that there is a certain probability of packet loss when the rtsp large bit rate is

## update_20200823
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

## update_20200504
* Fixed the issue that NDI authorization will be cleared after the upgrade

## update_20200504
* Added remote assistance function (bottom of system settings)

## update_20200502
* Correct the latency time unit of SRT
* Open 3520D watchdog

## update_20200430
* Added 4G module support
* Fixed the problem of abnormal reconnection when opening multiple SRT channels

## update_20200427
* Fix the wifi management page

## update_20200426
* Modify the SRT user interface
* Added wifi module support
* Fix the exception of Overlay addition, deletion and modification

## update_20200420
* Newly added platform live video preview
* Improve the applet interface
