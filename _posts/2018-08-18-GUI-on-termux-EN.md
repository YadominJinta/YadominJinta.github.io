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
As we know, there isn't X package in Termux repository. To install X, we need to enable the community repository  _[Termux Extra Package](https://github.com/xeffyr/termux-extra-packages)_.  

``` bash
# Install dependencies 
pkg in dirmngr 
# Add key
apt-key adv --keyserver pool.sks-keyservers.net --recv 9D6D488416B493F0 
# Add repository
echo "deb https://termux.xeffyr.ml/ extra main x11" >> $PREFIX/etc/sources.list 
# Update database
pkg up 
# Install GUI packages
# I will use tigervnc as the server , i3 as the wm
pkg in tigervnc i3 aterm
```

### Start VNC
```
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

现在我们打开之前下载的VNC Viewer，点击右下角的+号，在Address下面填上127.0.0.1:1，Name 随便填就行，不影响，点击Create -> Connect，会出现一个不安全链接的Warn，去掉Warn Me Everytime，点击OK，然后输入密码，点击Remember，然后Ok，就可以链接上了。  
Now open _VNC Viewer_, click the plus icon. 
![vnc1](/assets/img/vnc1.png)  
![vnc2](/assets/img/vnc2.png)  

