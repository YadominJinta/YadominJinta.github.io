---
date: 2019-10-16
title: 学习使用nginx
cover: https://nginx.org/nginx.png
tags: Linux
layout: post
---


Nginx是一款轻量级的Web服务器、反向代理服务器，由于它的内存占用少，启动极快，高并发能力强，在互联网项目中广泛应用。

# 安装

各大发行版都提供了nginx的软件包，不过对于Debian等发行版可能需要自己添加PPA源以获取nginx的最新版

``` bash
dnf install nginx
# On Fedora
```

# 启动Nginx

Nginx也使用`systemd`进行管理。

``` bash
systemctl enable nginx
# 让nginx开机自启
systemctl start nginx
# 运行nginx
```

# 配置nginx

首先贴出来ngnix的默认配置

``` nginx
cat /etc/nginx/nginx.conf

# For more information on configuration, see:
#   * Official English Documentation: http://nginx.org/en/docs/
#   * Official Russian Documentation: http://nginx.org/ru/docs/

user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

# Load dynamic modules. See /usr/share/doc/nginx/README.dynamic.
include /usr/share/nginx/modules/*.conf;

events {
    worker_connections 1024;
}

http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 2048;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    # Load modular configuration files from the /etc/nginx/conf.d directory.
    # See http://nginx.org/en/docs/ngx_core_module.html#include
    # for more information.
    include /etc/nginx/conf.d/*.conf;

    server {
        listen       80 default_server;
        listen       [::]:80 default_server;
        server_name  _;
        root         /usr/share/nginx/html;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location / {
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
}
```

让我们来分析一下这个conf。

``` nginx
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;
```

- `error_log`， nginx的错误日志  
- `pid`, PID file，检测进程状态

``` nginx
events {
    worker_connections 1024;
}
```
- `events`块:配置影响nginx服务器或与用户的网络连接。有每个进程的最大连接数，选取哪种事件驱动模型处理连接请求，是否允许同时接受多个网路连接，开启多个网络连接序列化等。
- `work_connections 1024;` 设置最大连接数为1024

``` nginx
http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 2048;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    # Load modular configuration files from the /etc/nginx/conf.d directory.
    # See http://nginx.org/en/docs/ngx_core_module.html#include
    # for more information.
    include /etc/nginx/conf.d/*.conf;

    server {
            ...
        }
    }
}
```

- `http`块：可以嵌套多个server，配置代理，缓存，日志定义等绝大多数功能和第三方模块的配置。如文件引入，mime-type定义，日志自定义，是否使用sendfile传输文件，连接超时时间，单连接请求数等。
- `include             /etc/nginx/mime.types;` 文件扩展名与文件类型映射表
- `default_type        application/octet-stream;` 默认文件类型
- `sendfile            on;` 允许用sendfile的方式传输文件
- `include /etc/nginx/conf.d/*.conf;` 从/etc/nginx/conf.d/中读取文件
- `keepalive_timeout   65;`  连接超时时间，默认为75s，可以在http，server，location块。
- `sendfile_max_chunk 100k;`  每个进程每次调用传输数量不能大于设定的值，默认为0，即不设上限。

``` nginx
server {
    listen       80 default_server;
    listen       [::]:80 default_server;
    # 监听端口
    server_name  _;
    root         /usr/share/nginx/html;

    # Load configuration files for the default server block.
    include /etc/nginx/default.d/*.conf;

    location / {
    }

    error_page 404 /404.html;
        location = /40x.html {
    }
    # 40X 页面

    error_page 500 502 503 504 /50x.html;
        location = /50x.html {
    }
    # 50X 页面
}
```

- `server`块：配置虚拟主机的相关参数，一个http中可以有多个server。

## 配置详解

### 主机与端口

``` nginx
listen 127.0.0.1:8000;
listen *:8000;
listen localhost:8000;
# IPV6
listen [::]:8000;
# other params
listen 443 default_serer ssl;
listen 127.0.0.1 default_server accept_filter=dataready backlog=1024
```

### 服务域名

``` nginx
# 支持多域名配置
server_name www.barretlee.com barretlee.com;
# 支持泛域名解析
server_name *.barretlee.com;
# 支持对于域名的正则匹配
server_name ~^\.barret\.com$;
```

### URL匹配

``` nginx
location = / {
    # 完全匹配  =
    # 大小写敏感 ~
    # 忽略大小写 ~*
}
location ^~ /images/ {
    # 前半部分匹配 ^~
    # 可以使用正则，如：
    # location ~* \.(gif|jpg|png)$ { }
}
location / {
    # 如果以上都未匹配，会进入这里
}
```

## 文件配置

### 根目录

``` nginx
location / {
    root /home/barret/test/;
}
```

### 别名

``` nginx
location /blog {
    alias /home/barret/www/blog/;
}
location ~ ^/blog/(\d+)/([\w-]+)$ {
    # /blog/20141202/article-name  
    # -> /blog/20141202-article-name.md
    alias /home/barret/www/blog/$1-$2.md;
}
```

### 首页

``` nginx
index /html/index.html /php/index.php;
```

### 重定向页面

``` nginx
error_page    404         /404.html;
error_page    502  503    /50x.html;
error_page    404  =200   /1x1.gif;
location / {
    error_page  404 @fallback;
}
location @fallback {
    # 将请求反向代理到上游服务器处理
    proxy_pass http://localhost:9000;
}
```

### Try_files

``` nginx
try_files $uri $uri.html $uri/index.html @other;
location @other {
    # 尝试寻找匹配 uri 的文件，失败了就会转到上游处理
    proxy_pass  http://localhost:9000;
}
location / {
    # 尝试寻找匹配 uri 的文件，没找到直接返回 502
    try_files $uri $uri.html =502;
}
```

## 反向代理

反向代理在计算机网络中是代理服务器的一种。服务器根据客户端的请求，从其关系的一组或多组后端服务器（如Web服务器）上获取资源，然后再将这些资源返回给客户端，客户端只会得知反向代理的IP地址，而不知道在代理服务器后面的服务器集群的存在。

与前向代理不同，前向代理作为客户端的代理，将从互联网上获取的资源返回给一个或多个的客户端，服务端（如Web服务器）只知道代理的IP地址而不知道客户端的IP地址；而反向代理是作为服务器端（如Web服务器）的代理使用，而不是客户端。客户端借由前向代理可以间接访问很多不同互联网服务器（集群）的资源，而反向代理是供很多客户端都通过它间接访问不同后端服务器上的资源，而不需要知道这些后端服务器的存在，而以为所有资源都来自于这个反向代理服务器。

反向代理在现时的互联网中并不少见，而另一些例子，像是CDN、SNI代理等，是反向代理结合DNS的一类延伸应用。

![](/assets/img/nginx1.png)


<p align="right">来源：Wikepedia</p>

下面以`Jekyll Blog`为例，在`http`块中加入

``` nginx
upstream jekyll {
    server 127.0.0.1:4000;
}
```

同时加入`location`

``` nginx
location / {
    proxy_pass_header Serve r;
    proxy_set_header Host $http_host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Scheme $scheme;
    proxy_pass http://jekyll;
}
```

然后这里踩了一个天坑，`SELinux`

``` bash
setsebool -P httpd_can_network_connect 1
# 允许http访问
```

然后启动jekyll
``` bash
jekyll s
```

打开浏览器看看效果

![](/assets/img/nginx2.png)

OK，先扯到这里，剩下负载均衡这些用得到再写写看看。