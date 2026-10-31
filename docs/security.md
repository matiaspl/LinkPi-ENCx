# Security findings and hardening

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

## Secure empty password system accounts
To disable the possibility of logging in without a password over telnet, change hashes for users other than root to `:x:` (for example using _vi_). You should end up with the following:

```
(...)
messagebus:x:1000:1000:Linux User,,,:/home/messagebus:/bin/sh
avahi:x:1002:1002:Linux User,,,:/home/avahi:/bin/sh
netdev:x:1003:1003:Linux User,,,:/home/netdev:/bin/sh
```

## Disable telnet daemon (OUTDATED, for recent versions look below)
Comment the _telnetd_ line from `/etc/init.d/rcS`.

## Disable non-secure network services (e.g. telnet, onvif)
Edit /link/config/service.json (thanks to [@Gradinko](https://github.com/Gradinko)) :

 /link/config # more service.json
 { "telnet":false, "ssh":true, "php":true, "nginx":true, "crond":true, "onvif":true, "ndi":true, "sls":true, "frp":false, "trans":false }
