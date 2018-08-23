---
layout: post
date: 2018-08-22
title: 在Termux上使用QEMU
tag: termux
---
几年前，我在安卓上折腾Win虚拟机的时候，试了各种各样奇怪的存在，包括[Bochs](https://play.google.com/store/apps/details?id=net.sourceforge.bochs)和[Limbo](https://play.google.com/store/apps/details?id=fr.energycube.android.app.com.limbo.emu.main.armv7)，以及贴吧的APQ，但都觉得不太好用。现在发现Termux下也有能跑虚拟机的，也就是大名鼎鼎的[QEMU](https://github.com/qemu/qemu)，上面提到的几个软件，其实都是基于QEMU。撒，就让我来介绍一下吧。

## 添加仓库

与上一篇[在Termux上使用图形化](https://yadominjinta.github.io/2018/07/30/GUI-on-termux.html)
遇到的问题相同，Termux的官方源中也没有QEMU，不过这次我们要换一个社区源[It's Pointless](https://wiki.termux.com/wiki/Package_Management#By_its-pointless_.28live_the_dream.29:)源。

``` bash
pkg install curl
curl -fsSL https://its-pointless.github.io/setup-pointless-repo.sh | bash
rm pointless.gpg
```
## 安装
这就很简单了
``` bash
pkg in qemu-system-x86
qemu-system-x86_64 --version

QEMU emulator version 2.12.0
Copyright (c) 2003-2017 Fabrice Bellard and the QEMU Project developers

```

## 下载镜像
前面说了啊，Bochs那几个都是基于QEMU，所以这几个能用的镜像也能用在QEMU中。下面列出几个能找到镜像的地方。  
[QEMU吧](https://tieba.baidu.com/f?kw=qemu)  
[Bochs吧镜像分享贴](http://tieba.baidu.com/p/5822419828)  
[Limbo吧镜像商店](https://tieba.baidu.com/p/3256889059)  
至于百度云的下载速度，我相信你们能搞定的！我这里随便了找了个WinXP的镜像做演示。  

## 启动QEMU
首先以我这个镜像为例，
``` bash
$ qemu-system-x86_64 -hda WindowsXP.qcow2 -m 1024 -netdev user,id=user.0 -device rtl8139,netdev=user.0 -vga vmware
VNC server running on 127.0.0.1:5900
```
现在打开我们的老朋友_VNC Viewer_，新建一个Connection，与上次不同的是，这次我们填写地址为 `127.0.0.1:5900` ，然后连接。  
![QEMU1](/assets/img/qemu1.png)

## 解读
下面，我来一个个参数解释。  
 `-hda WindowsXP.qcow2` ：设置第一启动磁盘为_WindowsXP.qcow2_，这个qcow2是qemu的一种磁盘格式，动态占用。  

 `-m 1024` ：设置内存为1024M，默认是M，后缀可以是K,M,G,T,E。  

 `-netdev user,id=user.0` ：创建一个网络，模式为User，id为user.0。  

 `-device rtl8139,netdev=user.0` ：使用网卡rtl8139，连接user.0。_rtl8139_是一种网卡，是不是用这个网卡取决于你的镜像，可以问问镜像的作者，使用 `qemu-system-x86_64 -net nic,model=?` 来查看所有支持的网卡。  

 `-vga vmware` ：使用vmware的显卡，与上面的网卡相同，都需要你去了解一下镜像作者选用的具体型号。  

暂时就写到这里吧，快要开学了，我很绝望。


