---
layout: post
title: Using GUI on termux
date: 2018-08-18
tags: termux
---

I know many of you want to use GUI on termux, so I write this!

## Native termux
Actually,it's impossible to use GUI in termux's windows. To use GUI, you need to prepare [VNC Viewer](https://play.google.com/store/apps/details?id=com.realvnc.viewer.android) or [XServer XSDL](https://play.google.com/store/apps/details?id=x.org.server), I will use VNC Viewer here.

### Add repository
As we know, there isn't X package in Termux repository. To install X, we need to open X repo.

``` bash
# Enable Repo
pkg in x11-repo
# Update database
pkg up 
# Install GUI packages
# I will use tigervnc as the server , i3 as the wm
pkg in tigervnc i3 aterm
```

### Start VNC
```bash
# Defind DISPLAY, better add to .bashrc
export DISPLAY=:1

$ vncserver :1                      
# Port :1
You will require a password to access your desktops.
# Input your passed here 
Password:
Verify:
Would you like to enter a view-only password (y/n)? n
# Useless for most of us.
New 'localhost:1 (u0_a385)' desktop is localhost:1

Creating default startup script /data/data/com.termux/files/home/.vnc/xstartup
Creating default config /data/data/com.termux/files/home/.vnc/config
Starting applications specified in /data/data/com.termux/files/home/.vnc/xstartup
Log file is /data/data/com.termux/files/home/.vnc/localhost:1.log

# If you need to use i3wm, please vim ~/.vnc/xstartup
---  twm &
+++  i3-wm &

# Restart Server
killall Xvnc
vncserver :1
# You may meet XLock problem when restarting
rm -rf $PREFIX/tmp/.X*
```


Now open _VNC Viewer_, click the plus icon. Write `127.0.0.1:1` in address ,click Create -> Connect -> unset Warn Me everytime -> OK, then input your password, click remember and ok. It's connect now.
![vnc1](/assets/img/vnc1.png)  
![vnc2](/assets/img/vnc2.png)  

## Non-Native

You must notice that there aren't many GUI applications in extra repository. But we can use proot to run a full linux distro, and use that.

But ,how can we install that ? You can install it with my script _[atilo](https://github.com/YadominJinta/atilo)_,use it as the instruction in README. For example,

``` bash
pkg in curl
curl https://raw.githubusercontent.com/YadominJinta/atilo/master/atilo -o ~/atilo 
chmod +x atilo
# Download atilo
./atilo install fedora
# Use it to install fedora
startfedora
# Boot fedora
```

To use GUI , you can do as the followings,(For Fedora and Debian based)

``` bash
# For Fedora
$ startfedora
dnf makecache
dnf install tigervnc-server 
dnf groupinstall LXDE
# for Debian
$ startdebian
apt update
apt install --no-install-recommands tigervnc-standalone-server lxde

# Boot Vnc
vncserver :1
# The same as the above

# Fedora
vim ~/.vnc/xstartup
--- exec /etc/X11/xinit/xinitrc
+++ exec startlxde
killall Xvnc
rm -rf /tmp/X1*
vncserver :1

# Debian 
vim /etc/X11/Xvnc-session
--- exec /etc/X11/Xsession "$@"
+++ exex startlxde
killall Xtigervnc # Notice that the process name is Xtigervnc, different with Fedora
rm -rf /tmp/.X1*
```

![vnc3](/assets/img/vnc3.png)  

![vnc4](/assets/img/vnc4.png)  

 That's all ,thank you for reading..
