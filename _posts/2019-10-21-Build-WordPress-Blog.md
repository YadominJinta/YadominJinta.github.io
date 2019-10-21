---
layout: post
tag: Linux
title: 搭建WordPress博客
date: 2019-10-21
cover: https://i.loli.net/2019/10/21/Zljy235T1wpkaKA.png
---

今天，想搭建一个属于自己的博客是相当容易的事，用`Jekyll`/`Hexo` + `Github Pages`十几分钟就可以搭建出来属于你自己的Blog。但是这些静态博客也有自己的缺陷，比如文章不能设置密码以限制访问(HTML SQL：你在说什么？)，所以今天我们来用WordPress搭建一个动态博客。

# 服务器与域名

服务器不用说了，看自己的需求买吧。

域名可以用Freenom的免费域名，然后使用CloudFlare的DNS解析服务，服务器在境外并且访问速度比较慢还可以用CF的反代来加速。

# LNMP

LNMP一键安装包是一个用Linux Shell编写的可以在多种发行版上安装的Linux程序。无需一个一个的输入命令，无需值守，编译安装优化编译参数，提高性能，解决不必要的软件间依赖，特别针对配置自动优化。

> 截止文章发布，LNMP的最新版本为1.6，具体见[LNMP](https://lnmp.org)

## 安装

``` bash
curl -O http://soft.vpser.net/lnmp/lnmp1.6.tar.gz
tar xf lnmp*.tar.gz
cd lnmp*/
./install.sh
# 然后一路回车（不过也可以修改一些内容）
