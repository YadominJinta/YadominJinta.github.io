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
```
现在我们打开之前下载的VNC Viewer，点击右下角的+号，在Address下面填上127.0.0.1:1，Name 随便填就行，不影响，点击Create -> Connect，会出现一个不安全链接的Warn，去掉Warn Me Everytime，点击OK，然后输入密码，点击Remember，然后Ok，就可以链接上了。  
![vnc1](/assets/img/vnc1.png)  
![vnc2](/assets/img/vnc2.png)  
  

## 非原生
想必你一定已经注意到了，Extra源里实在是没什么可以玩的，没有什么GUI应用。那么我们还有另一种玩法，用proot启动正常的Linux发行版，在Linux里面启动vnc。  
但是，怎么安装别的Linux发行版呢？  
这里我们可以用我正在维护的[atilo](https://github.com/YadominJinta/atilo)，按照README中的说明，安装一个发行版，这里以我最喜欢的Fedora28为例。  

```
$ startfedora
# dnf makecache
# dnf install tigervnc-server 
```
