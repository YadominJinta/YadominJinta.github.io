---
layout: post
title: 使用Grub启动ISO
date: 2018-04-01
tags: [Linux]
---

近期作死在服务器里换系统，客户回复实在太慢，最后自己动手，丰衣足食，所以我选择用Grub(Grub2)来启动安装镜像。以下分享一些我成功的经验。

## 通用

```
menuentry "Name" {
set root=(hdx,y)
set isofile='/isofile.iso'
set loader='/foo'
loopback loop $isofile
linux (loop)$loader/linux args
initrd (loop)$loader/initrd
}
```
解释

`root` :ISO文件所在盘  

`isofile` :ISO文件所在目录  

`loader`:ISO文件中linux和initrd所在目录  

`linux`:加载内核的命令，后面的`args`为内核参数  

建议打开ISO文件，从里面找到grub.cfg，从里面找到`loader`和`args`，其中`args`可能需要做一些修改。  

另外，大部分人跟我一样，都只有一个分区是可以被Grub读取的，所以这里我尽量选择网络的方法。 

以下为我尝试的一些发行版的启动方法。

## openSuSE

```
menuentry "openSUSE" {
  load_video
  insmod gzio
  insmod part_msdos
  insmod ext2
  set root='(hdx,y)'
  set isofrom_device='/dev/sdax'
  set isofrom_system='/opensuse.iso'
  set loader='/boot/x86_64/loader'
  loopback loop $isofrom_system
  linux   (loop)$loader/linux isofrom_device=$isofrom_device isofrom_system=$isofrom_system ramdisk_size=512000 ramdisk_blocksize=4096 ro quiet splash $vt_handoff preloadlog=/dev/null showopts
  initrd  (loop)$loader/initrd
}
```
注意，openSuSE在国内网络安装时可能遇到各种迷之验证错误。  
使用的镜像为openSUSE-Tumbleweed-NET-x86_64-Current.iso。

## Fedora

```
menuentry 'Fedora' {
    set isofile='/fedora.iso'
    set loader='/isolinux'
    set base_url='http://mirrors.tuna.tsinghua.edu.cn/fedora/releases'
    set release='27'
    set fversion='Everything'
    loopback loop $isofile
    linux (loop)$loader/vmlinuz inst.stage2=$base_url/$release/$fversion/x86_64/os rootfstype=vfat quiet rhgb 
    initrd (loop)/isolinux/initrd.img
}
```

### 解释
`base_url`:网络安装的镜像网址

`release`:Fedora的版本，这里是27

`fversion`:Fedora的不同镜像，这里是Everything网络安装版.  

使用镜像为Fedora-Everything-netinst-x86_64-27-1.6.iso




以后**有时间**一定会增加
