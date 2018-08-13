---
layout: post
title: '像用Linux一样用MacOS（1）'
date: 2017-09-23
tags: macOS
categories: 技术
---
因为种种原因，我从Linux转向了黑苹果，毕竟同为Unix Like系统，本以为二者用法基本相同，但是体验后才发现，这两个系统的差别真的不小，所以我想尽力把MacOS用成Linux。  

## 安装一个包管理器
用过Linux的朋友们都清楚，包管理器是一个发行版最大的特征，无论是Debian系的apt和dpkg，Red Hat系的yum,dnf和rpm，还是Arch系的pacman，SUSE系的zypper，都是很优秀的包管理器，有自己的优势与长处。既然想要像Linux，那么一个包管理器是不可少的，但是，MacOS上有包管理器吗？App Store算吗？当然不算！

这里介绍一下MacOS上第三方包管理器，[Brew](https://brew.sh/index_zh-cn.html),安装的话，直接执行
``` bash
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```
就可以在你MacOS上安装上Brew了，现在，输入`brew install gcc`，就可以在MacOS安装gcc，不过这个Gcc没有Linux用起来那么容易，我在后面的文章会介绍的。  

## 其他包管理器
除了Brew以外，还有许多跨平台的包管理器也可以在MacOS上安装，譬如我们熟知的pip,gem和npm。可以直接使用brew安装.
``` bash
brew install python3  
pip3 install Netease-Musicbox
```
然后就安装好了巨好用的网易云命令行客户端，`musicbox`,![mac1.1](/assets/img/mac1.1.png)
