---
layout: post
title: 破解Aria2的线程限制
date: 2017-10-02
tags: macOS aira2 Linux
---
Aria2是一个非常好的下载器，不管在那个平台上，都以快著称，也有许多插件使用了Aria2，比如那个用来破解百度云限速的[BaiduExport](https://github.com/acgotaku/BaiduExporter),但是Aria2有个令人不快的地方，那就是16线程的限制，虽然可以用一个叫做[axel](https://github.com/axel-download-accelerator/axel)的软件,但是axel没有RPC服务器一类的功能，所以还是Aria2，这么一来，我想要重新编译aria2。

## 准备
注：Arch用户请直接yaourt -S aria2-fast或者导入cn源后pacman -S aria2-fast

首先下载Aria2最新的[Release](https://github.com/aria2/aria2/releases/download/release-1.32.0/aria2-1.32.0.tar.xz),因为某些原因，可能需要科学上网才能下载。至于为什么不用直接用git呢，因为github上最新的源码需要准备很多的依赖来生成正确的configure，而release里面已经弄好了。  
测试之后发现，只需要有个gcc和g++或者clang（MacOS上只有clang :( ）就行，并不需要其他依赖。  

## 修改
下载源码后解压`tar xvf aria2*`然后进入目录aria*/src，找到OptionHandlerFactory.cc,`vim OptionHandlerFactory.cc`。  
接下来的内容请大家自行使用vim的搜索功能
将
``` cpp
OptionHandler* op(new NumberOptionHandler(PREF_MAX_CONNECTION_PER_SERVER,
                                               TEXT_MAX_CONNECTION_PER_SERVER,
                                               "1", 1, 16, 'x'));
```
修改为
``` cpp
OptionHandler* op(new NumberOptionHandler(PREF_MAX_CONNECTION_PER_SERVER,
                                               TEXT_MAX_CONNECTION_PER_SERVER,
                                               "128", 1, -1, 'x'));
```
将
``` cpp
PREF_MIN_SPLIT_SIZE, TEXT_MIN_SPLIT_SIZE, "20M", 1_m, 1_g, 'k'));
```
修改为
``` cpp
PREF_MIN_SPLIT_SIZE, TEXT_MIN_SPLIT_SIZE, "4K", 1_k, 1_g, 'k'));
```
将
``` cpp
PREF_CONNECT_TIMEOUT, TEXT_CONNECT_TIMEOUT, "60", 1, 600));
```
修改为
``` cpp
PREF_CONNECT_TIMEOUT, TEXT_CONNECT_TIMEOUT, "30", 1, 600));
```
将
``` cpp
PREF_PIECE_LENGTH, TEXT_PIECE_LENGTH, "1M", 1_m, 1_g));
```
修改为
``` cpp
PREF_PIECE_LENGTH, TEXT_PIECE_LENGTH, "4k", 1_k, 1_g));
```
将
``` cpp
new NumberOptionHandler(PREF_RETRY_WAIT, TEXT_RETRY_WAIT, "0", 0, 600));
```
修改为
``` cpp
new NumberOptionHandler(PREF_RETRY_WAIT, TEXT_RETRY_WAIT, "2", 0, 600));
```
将
``` cpp
new NumberOptionHandler(PREF_SPLIT, TEXT_SPLIT, "5", 1, -1, 's'));
```
修改为
``` cpp
new NumberOptionHandler(PREF_SPLIT, TEXT_SPLIT, "8", 1, -1, 's'));
```

好了，到这里为止我们全部修改完了，然后保存退出就行。

## 编译
这部分没什么好讲的，进入aria2的目录  

在Fedora测试后，需要
``` bash
sudo dnf install openssl openssl-devel libssh2 libssh-devel libxml2 libxml2-devel c-ares c-ares-devel libsqlite3x libsqlite3x-devel
```
才能够开启全部功能，其他发行版请自行尝试，包名应该差不多，Ubuntu上一般是xx-dev的头文件包(Arch:头文件包是啥).
``` bash
./configure --with-libxml2
make -j n
sudo make install
```
~~就是这么简单，这也是为什么我要选择Release的原因。~~  

这里写错了，并不是什么依赖都不需要，需要开启全部功能必须有正确的头文件，前不久回到Linux才注意到这个问题，因为用的时候发现不支持openssl，Macos上我是之前配置好的。  


测试一下
``` bash
aria2c -x 256 http://124.228.42.246/tech.down.sina.com.cn/20170601/48759a2b/GoogleChrome.dmg?fn=&ssig=cq0ei0QbzC&Expires=1506993310&KID=sae,230kw3wk15&ip=1506914110,36.5.54.145&corp=1
```
嗯，再也不会出现16线程的报错了。

## 其他
对于Archlinux用户，正如前文所述，直接安装aria2-fast  

对于macOS用户，如果你正在使用炒鸡好用的Aria2GUI的话，你一定发现线程只能开到32，所以想更高的话，请在按照上文编译完成后。  
打开Finder，应用程序，Aria2GUI，右击，显示包内容，进入Contents／Resources,将Aria2/src目录里编译出来的aria2c复制到改目录下，替换原来的。就可以无限的开线程了。
![aria2](/assets/img/aria21.png)
