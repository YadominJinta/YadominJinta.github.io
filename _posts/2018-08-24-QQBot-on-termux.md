---
date: 2018-08-24
title: 在termux上玩QQ机器人
layout: post
tag: termux
---

QQ机器人是个好玩的东西，也有很多解决方案，比如最著名的[酷Q机器人](https://cqp.cc/)
，到这个只能在PC上用。安卓上能用的[Sq QQ机器人](https://www.coolapk.com/apk/com.specher.qqrobot)，但是这个要Xposed，虽然基于手机qq注入，功能比较强，但是对我这种连Root都没有的用户，emmmm。所以我就来介绍**几个**能在Termux上玩的QQ机器人。

## [qqbot](https://github.com/pandolia/qqbot)
_Writen in Python，基于WebQQ协议_ 

### 安装
``` bash
pkg in python python-dev  clang# 安装依赖
pip3 install qqbot # 安装qqbot
pip3 install wcwidth
env LDFLAGS="-L/system/lib/ -lm -lcompiler_rt" pip3 install Pillow --global-option="build_ext" --global-option="--disable-jpeg"
# 安装后续用的依赖
```
### 启动
``` bash
# 安装以后自然要赶快启动看看喽，但是
$ qqbot
.......
pytz.exceptions.UnknownTimeZoneError: 'Can not find any timezone configuration'
# 如何解决呢？
$ vim $PREFIX/lib/python3.6/site-packages/tzlocal/unix.py
# 编辑tzlocal包的unix.py文件，在vim里面输入50gg，跳转到第50行，然后粘贴如下内容

if os.path.exists('/system/bin/getprop'):
 import subprocess 
androidtz=subprocess.check_output(['getprop', 'persist.sys.timezone']).strip().decode() 
return pytz.timezone(androidtz)

# 然后再启动，依然爆炸
$ qqbot
.......
pytz.exceptions.UnknownTimeZoneError: 'Can not find any timezone configuration'

# 又是怎么回事呢？
# 这是因为Termux的默认PATH不包括/system/bin/，加到PATH里面就好了

$ PATH=$PATH:/system/bin/

# 这下终于好了
qqbot -cq
# 以文本方式显示二维码，双指缩小字体使其能完整的显示，截图使用QQ扫码登陆。
```
![QQBot1](/assets/img/qqbot1.png)

### 使用
现在，从屏幕左向右滑动，点击New Session，创建一个新的终端
``` bash
$ qq help

QQBot 命令：
1） 帮助、停机和重启命令
    qq help|stop|restart

2） 联系人查询命令
    qq list buddy|group|discuss [qq|name|key=val]
    qq list group-member|discuss-member oqq|oname|okey=oval [qq|name|key=val]

3） 联系人更新命令
    qq update buddy|group|discuss
    qq update group-member|discuss-member oqq|oname|okey=oval

4） 消息发送命令
    qq send buddy|group|discuss qq|name|key=val message

5） 群管理命令： 设置/取消管理员 、 设置/删除群名片 、 群成员禁言 以及  踢除群成员
    qq group-set-admin ginfo minfo1,minfo2,...
    qq group-unset-admin ginfo minfo1,minfo2,...
    qq group-set-card ginfo minfo1,minfo2,... card
    qq group-unset-card ginfo minfo1,minfo2,...
    qq group-shut ginfo minfo1,minfo2,... [t]
```
基本使用方法如上，接下来实际使用看看
```
$ qq list buddy
好友列表：
+------+-------+----------+----------+--------+--------+------------+----------+
| 类型 | QQ    | 名称     | 网名     | 备注名 | 群名片 | UIN        | 群内角色 |
+------+-------+----------+----------+--------+--------+------------+----------+
| 好友 | #NULL | 高坂桐乃 | 高坂桐乃 |        |        | 1173114394 |          |
+------+-------+----------+----------+--------+--------+------------+----------+

$ qq send buddy 高坂桐乃 你好
向 好友“高坂桐乃” 发消息成功
```
看看效果  
![QQBot2](/assets/img/qqbot2.png)

### 插件
可能你要问了，这个机器人就这点功能吗？  
很不幸，因为基于WebQQ协议(你能在Github上找到的，基本上都是WebQQ协议），功能确实孱弱了一些，要想扩展，我们可以自己来写插件，我这里提供一个图灵机器人的插件。
``` bash
pip3 install request
vim ~/.qqbot-tmp/plugins/turling.py
```
加入以下内容
``` python
import qqbot
 from random import randint
 import requests


 # 使用前请先前往 http://www.tuling123.com/register/index.jhtml
 # 申请 API key 谢谢
 # 另外需要 requests 支持
 # 修改成调用图灵官方接口
 url = 'http://www.tuling123.com/openapi/api'
 apikey = ''
# 请在上面填入你自己的API Key
 def onQQMessage(bot, contact, member, content):
     if bot.isMe(contact, member) != True:
         querystring = {
             "key": apikey,
             "info": content,
         }

         response = requests.request("GET", url, params=querystring)

         response_json = response.json()
         bot.SendTo(contact, response_json.get('text'))
```
_我完全没有Py基础，写的不好不要吐槽_  
然后使用`qq plug turling`来加载插件，效果如下。  
![qqbot3](/assets/img/qqbot3.png)  
  
这个项目大抵也就如此了，要想实现更多功能，还请自己写插件。  

## [Mojo-Webqq](https://github.com/sjdy521/Mojo-Webqq)
_Written in Perl，WebQQ协议_  
看到这里你是不是想吐槽，怎么还是WebQQ，我也很无奈啊，能找到的大都是WebQQ协议的，我倒是看到过[PCQQ](https://github.com/luojinfang/PCQQ-Protocol)协议的，但是C#写的，在Termux上有些难以运行。不过，这个Mojo-WebQQ在功能上还是很强的，至少官方提供了不少插件，下面容我细细道来。  

### 安装
``` bash 
pkg in wget make clang perl openssl-dev openssl-tool libcrypt libcrypt-dev
# 安装依赖
cpan -i App::cpanminus
# 安装另一个包管理器cpanm
cpanm Mojo::Webqq
# 安装Mojo-Webqq，可能会出现一些依赖问题，懒得把Termux重装一遍来测试了，如果有问题的话请跟我反馈。
cpanm Webqq::Encryption
# 安装用来密码登陆的库，但是我每次都失败😤
```
### 启动
编辑启动脚本
``` bash
vim login.pl
use Mojo::Webqq;
use Digest::MD5 qw(md5_hex);
my $client=Mojo::Webqq->new(
    account     => 123456,      #QQ账号
    pwd         => md5_hex('你的密码'),  #登录密码
    http_debug  =>  0,          #是否打印详细的debug信息
    log_level   =>  "info",     #日志打印级别，debug|info|msg|warn|error|fatal
    login_type  =>  "login",    #登录方式，login 表示账号密码登录
);

$client->load("ShowMsg");
$client->run();
```
`perl login.pl`来启动  
然后新建一个Session，输入`cp $TMPDIR/mojo*.png ~/storage/dcim/`，然后在qq里面扫码登陆，不过我这边在qq里面显示不出来这张图，所以我是在系统相册里再截一次图的。  

![qqbot5](/assets/img/qqbot5.png)  








