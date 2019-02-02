---
layout: post
date: 2018-07-30
tags: termux
title: 在termux上使用图形化
cover: https://i.loli.net/2019/02/02/5c54e434c82f6.png
---
最近老是被人问怎么在termux上用图形化(GUI)，想来不如写一篇文章得了。
~~被人吐槽写的太抽象了，我改还不行吗~~
# VNC
## Termux原生
准确来说，原生是不可能的，你不可能在termux那个窗口里用GUI的，你需要准备[VNC Viewer](https://play.google.com/store/apps/details?id=com.realvnc.viewer.android)或者[XServer XSDL](https://play.google.com/store/apps/details?id=x.org.server)，这里以VNC Viewer为例。(据称VNC这个太麻烦，可以看后文的XSDL使用介绍)

### 添加仓库
Termux的源中是没有X的，所以为了安装X，我们在这里添加X11的仓库
``` bash
# 添加仓库
pkg in x11-repo
# 更新数据库
pkg up
# 安装图形化相关软件
# 这里以TigerVNC为接口，i3为wm
pkg in tigervnc i3 aterm
```

### 启动VNC
``` bash
# 定义DISPLAY变量，建议加到.bashrc里
$ export DISPLAY=:1

$ vncserver :1 -geometry 1920x1080                          
# 端口为：1，分辨率为1920x180
You will require a password to access your desktops.
# 在这里输入密码，六位以上，我一般直接123456
Password:
Verify:
Would you like to enter a view-only password (y/n)? n
# 应该没有这个需求
New 'localhost:1 (u0_a385)' desktop is localhost:1

Creating default startup script /data/data/com.termux/files/home/.vnc/xstartup
Creating default config /data/data/com.termux/files/home/.vnc/config
Starting applications specified in /data/data/com.termux/files/home/.vnc/xstartup
Log file is /data/data/com.termux/files/home/.vnc/localhost:1.log

# 如果你需要使用i3，请vim ~/.vnc/xstartup
---  twm &
+++  i3-wm &

# 你在重启VNC的时候可能会遇到关于X lock之类的报错，请
$ rm -rf $PREFIX/tmp/.X*
```
现在我们打开之前下载的VNC Viewer，点击右下角的+号，在Address下面填上127.0.0.1:1，Name 随便填就行，不影响，点击Create -> Connect，会出现一个不安全链接的Warn，去掉Warn Me Everytime，点击OK，然后输入密码，点击Remember，然后Ok，就可以链接上了。  
![vnc1](/assets/img/vnc1.png)  
![vnc2](/assets/img/vnc2.png)  


## 非原生
想必你一定已经注意到了，Extra源里实在是没什么可以玩的，没有什么GUI应用。那么我们还有另一种玩法，用proot启动正常的Linux发行版，在Linux里面启动vnc。  
但是，怎么安装别的Linux发行版呢？  
这里我们可以用我正在维护的[atilo](https://github.com/YadominJinta/atilo)，按照README中的说明，安装一个发行版，这里以我最喜欢的Fedora28为例。  
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

# 启动Vnc
vncserver :1
# 与上面相同
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
killall Xtigervnc # 这里进程名为Xtigervnc，与Fedora不同
rm -rf /tmp/.X1*
```

~~被人吐槽了，我加个Debian吧~~

![vnc3](/assets/img/vnc3.png)
![vnc4](/assets/img/vnc4.png)
~~Ok，到此为止~~

# XSDL
XSDL这个就相对简单多了
# Termux原生
``` bash
# In termux
pkg in openbox aterm

# 本来准备用i3，结果发现i3不用无线键盘
#几乎不能正常使用，所以改成了openbox

export DISPLAY=:0 
#建议加入~/.bashrc

# 然后打开XSDL，等待它启动
openbox 

```
然后再打开openbox就可以看到界面了，，，，，，个屁啊  
实际上你只能看到一个小小的鼠标(我第一次看的时候还以为没启动，当然你也可以配置openbox来修改背景图)，双指点击屏幕，应该会出现openbox的菜单，滑动手指移动鼠标，按返回键可以掉出键盘。  
![xsdl1](/assets/img/xsdl1.png)  
但是Termux上能用的GUI程序太少了，尤其是在不用外接键盘的情况下，特别难操作，所以还是换成proot系统玩吧。

## 非原生
首先要安装[Atilo](https://github.co/YadominJinta/atilo)，并通过其安装一个发行版，这里就不多赘述了。依然使用Fedora为例。  
``` bash
startdfedora

dnf groupinstall lxde-desktop
# 安装LXDE

export DISPLAY=:0
export PLUSE_SERVER=tcp:127.0.0.1:4172
# 定义显示和声音变量
# 打开XSDL，等待启动

startlxde
# 启动LXDE
```
![xsdl2](/assets/img/xsdl2.png)  
![xsdl3](/assets/img/xsdl3.png)  
  
总得来说，这XSDL确实有它的好处，配置简单，支持声音，但是在操作性上真的很差劲，这点确实比不上VNC，至于选哪个，见仁见智吧。  

鸽了好久的更新~~（其实只是旧瓶添新酒）~~终于发出来了，提前祝各位新年快乐！  

最后秀一把桌面(我PC的)，以说明KDE才是世界上最好看的桌面环境XD
![Desktop](/assets/img/desktop1.png)
