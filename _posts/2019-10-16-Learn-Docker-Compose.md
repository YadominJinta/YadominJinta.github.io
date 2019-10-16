---
title: 学习使用docker-compose
layout: post
date: 2019-10-16
tags: Linux
cover: https://i.loli.net/2019/10/16/CqarzTRwoJiu9be.png
---

当你的一个应用需要用到多个docker镜像时，你会怎么做，写一个bash/Python脚本吗？ 其实docker已经有工具可以来定义和运行复杂的应用，这就是`docker-compose`。

# 安装

`docker-compose`使用`Python`编写，已加入`Pypi`，所以安装只需要

``` bash
pip3 install docker-compose
```

# 使用

## 术语

- 服务(Service)：一个应用容器，实际上可以运行多个相同镜像的实例。
- 项目(Project)：由一组关联的应用容器组成的一个完整业务单元。

## 场景

最常见的项目是 web 网站，该项目应该包含 web 应用和缓存。

下面我们用 Python 来建立一个能够记录页面访问次数的 web 网站。

### Web应用

新建文件夹，在文件夹中创建`app.py`

``` python
from flask import Flask
from redis import Redis

app = Flask(__name__)
redis = Redis(host='redis', port=6379)

@app.route('/')
def hello():
    count = redis.incr('hits')
    return 'Hello World! 该页面已被访问 {} 次。\n'.format(count)

if __name__ == "__main__":
    app.run(host="0.0.0.0", debug=True)
```

### Dockerfile

文件夹中还需要`Dockerfile`

``` dockerfile
FROM python:3.6-alpine
ADD . /code
WORKDIR /code
RUN pip install redis flask
CMD ["python", "app.py"]
```

### docker-compose.yml

以及docker-compose.yml

``` yml
version: '3'
services:

  web:
    build: .
    ports:
     - "5000:5000"

  redis:
    image: "redis:alpine"
```

### 运行

``` bash
docker-compose up
```

现在访问本地的`5000`端口看看

![](/assets/img/compose1.png)

# 命令

### `build`

格式为 `docker-compose build [options] [SERVICE...]`。

构建（重新构建）项目中的服务容器。

服务容器一旦构建后，将会带上一个标记名，例如对于`web`项目中的一个`db`容器，可能是`web_db`。

可以随时在项目目录下运行`docker-compose build`来重新构建服务。

选项包括：

- `--force-rm` 删除构建过程中的临时容器。

- `--no-cache` 构建镜像过程中不使用 `cache`（这将加长构建过程）。

- `--pull` 始终尝试通过 `pull` 来获取更新版本的镜像。

### `up`

格式为`docker-compose up [options] [SERVICE...]`。

该命令十分强大，它将尝试自动完成包括构建镜像，（重新）创建服务，启动服务，并关联服务相关容器的一系列操作。

链接的服务都将会被自动启动，除非已经处于运行状态。

可以说，大部分时候都可以直接通过该命令来启动一个项目。

默认情况，`docker-compose up` 启动的容器都在前台，控制台将会同时打印所有容器的输出信息，可以很方便进行调试。

当通过 `Ctrl-C` 停止命令时，所有容器将会停止。

如果使用 `docker-compose up -d`，将会在后台启动并运行所有的容器。一般推荐生产环境下使用该选项。

默认情况，如果服务容器已经存在，`docker-compose up` 将会尝试停止容器，然后重新创建（保持使用 `volumes-from` 挂载的卷），以保证新启动的服务匹配 `docker-compose.yml` 文件的最新内容。如果用户不希望容器被停止并重新创建，可以使用 `docker-compose up --no-recreate`。这样将只会启动处于停止状态的容器，而忽略已经运行的服务。如果用户只想重新部署某个服务，可以使用 `docker-compose up --no-deps -d <SERVICE_NAME>` 来重新创建服务并后台停止旧服务，启动新服务，并不会影响到其所依赖的服务。

选项：

- `-d` 在后台运行服务容器。

- `--no-color` 不使用颜色来区分不同的服务的控制台输出。

- `--no-deps` 不启动服务所链接的容器。

- `--force-recreate` 强制重新创建容器，不能与 --no-recreate 同时使用。

- `--no-recreate` 如果容器已经存在了，则不重新创建，不能与 `--force-recreate` 同时使用。

- `--no-build` 不自动构建缺失的服务镜像。

- `-t, --timeout TIMEOUT` 停止容器时候的超时（默认为 10 秒）。

### `down`

此命令将会停止`up`命令所启动的容器，并移除网络

### `logs`

格式为`docker-compose logs [options] [SERVICE...]`。

查看服务容器的输出。默认情况下，`docker-compose` 将对不同的服务输出使用不同的颜色来区分。可以通过 `--no-color` 来关闭颜色。

该命令在调试问题的时候十分有用。

### config

验证 Compose 文件格式是否正确，若正确则显示配置，若格式错误显示错误原因。

### `ps`

格式为`docker-compose ps [options] [SERVICE...]`。

列出项目中目前的所有容器。

选项：

`-q` 只打印容器的 ID 信息。

### `stop`

格式为 `docker-compose stop [options] [SERVICE...]`。

停止已经处于运行状态的容器，但不删除它。通过 `docker-compose start` 可以再次启动这些容器。

选项：

`-t`, `--timeout TIMEOUT` 停止容器时候的超时（默认为 10 秒）。

### `rm`
格式为 `docker-compose rm [options] [SERVICE...]`。

删除所有（停止状态的）服务容器。推荐先执行 `docker-compose stop` 命令来停止容器。

选项：

`-f`, `--force` 强制直接删除，包括非停止状态的容器。一般尽量不要使用该选项。

`-v` 删除容器所挂载的数据卷。

这几个命令应该是最常用的了吧

# Compose模板文件

模板文件是使用 Compose 的核心，涉及到的指令关键字也比较多。但大家不用担心，这里面大部分指令跟`docker run`相关参数的含义都是类似的。

默认的模板文件名称为`docker-compose.yml`，格式为 YAML 格式。

``` YAML
version: "3"

services:
  webapp:
    image: examples/web
    ports:
      - "80:80"
    volumes:
      - "/data"
```

注意每个服务都必须通过`image`指令指定镜像或`build`指令（需要 Dockerfile）等来自动构建生成镜像。

如果使用 build 指令，在 Dockerfile 中设置的选项(例如：CMD, EXPOSE, VOLUME, ENV 等) 将会自动被获取，无需在 docker-compose.yml 中重复设置。

下面分别介绍各个指令的用法。

### build

指定`Dockerfile`所在文件夹的路径（可以是绝对路径，或者相对 `docker-compose.yml`文件的路径）。 Compose 将会利用它自动构建这个镜像，然后使用这个镜像。

``` YAML
version: '3'
services:

  webapp:
    build: ./dir
```

你也可以使用`context`指令指定`Dockerfile`所在文件夹的路径。

使用`dockerfile`指令指定`Dockerfile`文件名。

使用`arg`指令指定构建镜像时的变量。

``` YAML
version: '3'
services:

  webapp:
    build:
      context: ./dir
      dockerfile: Dockerfile-alternate
      args:
        buildno: 1
```

### command

覆盖容器启动后默认执行的命令。

``` YAML
command: echo "hello world"
```

### depends_on

解决容器的依赖、启动先后的问题。以下例子中会先启动`redis` `db` 再启动`web`

``` YAML
version: '3'

services:
  web:
    build: .
    depends_on:
      - db
      - redis

  redis:
    image: redis

  db:
    image: postgres
```
> 注意：web 服务不会等待 redis db 「完全启动」之后才启动。

### tmpfs

挂载一个 `tmpfs` 文件系统到容器。

``` YAML
tmpfs: /run
tmpfs:
  - /run
  - /tmp
```

### env_file

从文件中获取环境变量，可以为单独的文件路径或列表。

如果通过 `docker-compose -f FILE` 方式来指定 Compose 模板文件，则 `env_file` 中变量的路径会基于模板文件路径。

如果有变量名称与 `environment` 指令冲突，则按照惯例，以后者为准。

``` YAML
env_file: .env

env_file:
  - ./common.env
  - ./apps/web.env
  - /opt/secrets.env
```

环境变量文件中每一行必须符合格式，支持 # 开头的注释行。

``` bash
# common.env: Set development environment
PROG_ENV=development
```

### environment

设置环境变量。你可以使用数组或字典两种格式。

只给定名称的变量会自动获取运行 `Compose` 主机上对应变量的值，可以用来防止泄露不必要的数据。

``` YAML
environment:
  RACK_ENV: development
  SESSION_SECRET:

environment:
  - RACK_ENV=development
  - SESSION_SECRET
```

### healthcheck

通过命令检查容器是否健康运行。

``` YAML
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost"]
  interval: 1m30s
  timeout: 10s
  retries: 3
```

### image

指定为镜像名称或镜像 ID。如果镜像在本地不存在，Compose 将会尝试拉取这个镜像。

``` YAML
image: ubuntu
image: orchardup/postgresql
image: a4bc65fd
```

### pid

跟主机系统共享进程命名空间。打开该选项的容器之间，以及容器和宿主机系统之间可以通过进程 `ID` 来相互访问和操作。

``` YAML
pid: "host"
```

### ports

暴露端口信息。

使用宿主端口：容器端口 (HOST:CONTAINER) 格式，或者仅仅指定容器的端口（宿主将会随机选择端口）都可以。

``` YAML
ports:
 - "3000"
 - "8000:8000"
 - "49100:22"
 - "127.0.0.1:8001:8001"
```

_注意：当使用 HOST:CONTAINER 格式来映射端口时，如果你使用的容器端口小于 60 并且没放到引号里，可能会得到错误结果，因为 YAML 会自动解析 xx:yy 这种数字格式为 60 进制。为避免出现这种问题，建议数字串都采用引号包括起来的字符串格式。_

### secrets

存储敏感数据，例如 mysql 服务密码。

``` YAML
version: "3.1"
services:

mysql:
  image: mysql
  environment:
    MYSQL_ROOT_PASSWORD_FILE: /run/secrets/db_root_password
  secrets:
    - db_root_password
    - my_other_secret

secrets:
  my_secret:
    file: ./my_secret.txt
  my_other_secret:
    external: true
```

### ulimits

指定容器的 ulimits 限制值。

例如，指定最大进程数为 65535，指定文件句柄数为 20000（软限制，应用可以随时修改，不能超过硬限制） 和 40000（系统硬限制，只能 root 用户提高）。

``` YAML
  ulimits:
    nproc: 65535
    nofile:
      soft: 20000
      hard: 40000
```

### volumes

数据卷所挂载路径设置。可以设置为宿主机路径(HOST:CONTAINER)或者数据卷名称(VOLUME:CONTAINER)，并且可以设置访问模式 （HOST:CONTAINER:ro）。

该指令中路径支持相对路径。

``` YAML
volumes:
 - /var/lib/mysql
 - cache/:/tmp/cache
 - ~/configs:/etc/configs/:ro
如果路径为数据卷名称，必须在文件中配置数据卷。

version: "3"

services:
  my_src:
    image: mysql:8.0
    volumes:
      - mysql_data:/var/lib/mysql

volumes:
  mysql_data:
```

这些应该够用了吧

# 实战NextCloud

场景： NextCloud