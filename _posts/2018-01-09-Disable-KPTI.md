---
layout: post
title: 关闭KPTI补丁
date: 2018-01-09
tags: [KTPI]
---
近期，Intel的Meltdown及Spectre两个漏洞的问题持续发酵，不同系统也发布补丁以修复Meltdown（Spectre暂时无法修复），但这些补丁都会使性能不同程度的降低。本文将介绍如何关闭这个补丁。  

## Windows
首先，看看你的电脑有没有打开这个功能，按照微软官方的方法，用管理员打开Powershell。

```
# Install the PowerShell Module
PS> Install-Module SpeculationControl

# Run the PowerShell module to validate the protections are enabled

PS> # Save the current execution policy so it can be reset

PS> $SaveExecutionPolicy = Get-ExecutionPolicy

PS> Set-ExecutionPolicy RemoteSigned -Scope Currentuser

PS> Import-Module SpeculationControl

PS> Get-SpeculationControlSettings

PS> # Reset the execution policy to the original state

PS> Set-ExecutionPolicy $SaveExecutionPolicy -Scope Currentuser 
```
如果打开的话应该能看到
```
Windows OS support for branch target injection mitigation is present: True

Windows OS support for branch target injection mitigation is enabled: True
```

所以我们该如何关闭呢？  
这个功能是通过更新KB4056892来实现的，卸载它应该是有效的，但是治标不治本，以后Windows更新还是会装上的。  
所以以管理员打开Powershell，输入`regedit.exe`，先把当前的注册表备份下来，然后
```
reg add "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Session Manager\Memory Management" /v FeatureSettingsOverride /t REG_DWORD /d 3 /f
reg add "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Session Manager\Memory Management" /v FeatureSettingsOverrideMask /t REG_DWORD /d 3 /f
```

按照前文所述检查一下
```
Windows OS support for kernel VA shadow is present: True
Windows OS support for kernel VA shadow is enabled: False
```
Ok!

## Linux
Linux的话这个补丁是写进内核的，所以有没有开启就看你发行版的心情了。在Fedora上，4.11.14以后的内核都是打开。检查一下你的电脑。
```
$ dmesg |grep "page table"
[    0.000000] Kernel/User page tables isolation: enabled 
```
说明已经打开KPTI，那么如何关闭呢。  
```
# vim /etc/default/grub
...
GRUB_CMDLINE_LINUX="...... nopti"
...

# grub2-mkconfig -o /etc/boot/EFI/fedora/grub.cfg
##不同发行版grub.cfg的位置不同，请自行修改。
# reboot
```
现在再看看
```
$ dmesg |grep "page table"
[    0.000000] Kernel/User page tables isolation: disabled on command line.
```

Ok!

## 注
macOS那边由于好久没有碰过了，所以我也不清楚，升级到10.13.2就会开启那个补丁吧。