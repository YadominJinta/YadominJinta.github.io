---
layout: post
date: 2018-06-15
tags: Linux
title: 在Fedora28上安装网易云音乐
---

Fedora28升级以来，我一直没能找到打包好的网易云。  
在27的时候有[yelanxin/netease-cloud-music](https://copr.fedorainfracloud.org/coprs/yelanxin/netease-cloud-music/),但是作者一直没有更新到28，而且28在依赖上也有一些不同，直接安装27的包会出现依赖错误。所以应该怎么在Fedora28上安装网易云音乐呢？我想到了手动安装的方法。

```
# 打开仓库
sudo yum install -y --nogpgcheck https://mirrors.tuna.tsinghua.edu.cn/rpmfusion/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm https://mirrors.tuna.tsinghua.edu.cn/rpmfusion/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm
sudo dnf install -y fedora-workstation-repositories
sudo dnf config-manager --set-enabled google-chrome
sudo dnf mc

# 建立缓存文件夹
mkdir netease && cd netease

# 下载安装包
wget http://d1.music.126.net/dmusic/netease-cloud-music_1.1.0_amd64_ubuntu.deb

# 解压
ar -xvf *.deb
tar xvf data.tar.xz

# 安装文件
sudo cp usr/* /usr/

# 安装依赖
sudo dnf in vlc google-chrome
```

以上即安装方法，效果图

![netease](/assets/img/netease.png)

[亚洲天后周婕纶](http://music.163.com/#/mv?id=5288150),画风超赞！
