---
layout: post
date: 2018-07-30
tags: termux
title: 在termux上使用图形化
---
最近老是被人问怎么在termux上用图形化(GUI)，想来不如写一篇文章得了。

## Termux原生
准确来说，原生是不可能的，你不可能在termux那个窗口里用GUI的，你需要准备[VNC Viewer](https://play.google.com/store/apps/details?id=com.realvnc.viewer.android)或者[XServer XSDL](https://play.google.com/store/apps/details?id=x.org.server)，这里以VNC Viewer为例。

### 添加仓库
Termux的源中是没有X的，所以为了安装X，我们在这里添加一个社区源——[Termix-Extra-Packages](https://github.com/xeffyr/termux-extra-packages)
```
# 安装依赖
pkg in dirmngr
# 添加密钥
apt-key adv --keyserver pool.sks-keyservers.net --recv 9D6D488416B493F0
# 添加仓库
echo "deb https://termux.xeffyr.ml/ extra main x11" >> $PREFIX/etc/sources.list
# 更新数据库
pkg up
# 安装图形化相关软件
# 这里以TigerVNC为接口，i3为wm
pkg in tiger tigervnc i3 aterm
```

### 启动VNC
```
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
```
现在我们打开之前下载的VNC Viewer，点击
