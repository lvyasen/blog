---
title: Docker 相关命令
urlname: oo046ytv07pqbktf
date: '2022-12-29 09:57:22 +0800'
tags: []
categories: []
---

tags: [Docker 相关命令]
categories: [Docker]
cover: ''

---

> 获取帮助 `docker help`。

## docker 信息

- docker info：打印 Docker 系统和主机的各种信息。
- docker help：把一个子命令作为参数，打印有关该子命令的使用方法和帮助信息。相当于运行命令时提供 --help
- docker version：打印 Docker 客户端和服务器版本，以及编译时使用的 Go 版本。

## 镜像相关

> 官方搜索网址：[https://hub.docker.com/](https://hub.docker.com/)

- 本地主机镜像列表：`docker images`
- 搜索镜像：`docker search mysql`
  - --limit 3 限制条数
- 拉取镜像：`docker pull <镜像名>[:tag]`没有 tag 就是最新版
- 查看镜像占用资源：`docker system df`
- 删除镜像：`docker rmi -f <镜像名/镜像ID>`
- 删除全部：`docker rmi -f $(docker images -qa)`
- 保存镜像：`docker save <镜像名> -o /myimg.tar`
- 导出镜像：`docker export swoole_php -o /Users/mac/swoole_php.tar`
- 加载镜像：` docker load -i <镜像保存文件位置>`
- 从 Dockerfile 建立镜像：`docker build`
  - -t：指定标签。完整命令 `--tag`
- 修改 tag ：`docker tag test:latest test:new`

### 镜像制作

> [相关文档](https://blog.csdn.net/m0_53151031/article/details/123052743) > [本地镜像推送到阿里云](https://blog.csdn.net/mengfanshaoxia/article/details/127155020)

## 容器相关

- 查看运行的容器：`docker ps`
  - -a：全部容器。
  - -l：查看最近创建的容器。完整命令 `--latest`
  - -q：只显示容器 ID。完整命令 `--quiet`
- 进入容器：
  - `docker exec -it 容器 ID bash`一般用这种
  - `docker --attach 容器 ID`这种方式进入容器存在问题，当多个终端同时进入容器，所有窗口同步显示。
- 从容器中退出：`exit`。
- 停止容器：`docker stop <容器名/容器 ID>`
- 删除容器：`docker rm -f <容器名、容器 ID>` 多个用空格隔开
- 删除全部容器：`docker rm -f $(dockerps -aq)`
- 重启容器：`docker  restart <容器名/容器 ID>`
- 启动容器：`docker start <容器名/容器 ID>`
- 查看容器日志：`docker logs <容器名/容器 ID>`
  - -f：追踪日志，保持打开状态，实时查看日志。
  - --tail：查看末尾多少行。
- 创建容器：`docker run --name "容器名" 镜像名`
  - -i：交互模式运行容器，通常与 `-t`同时使用。完整为 `--interactive`意思是交互。
  - -t：启动容器后，为容器分配一个命令行；简写原为 `--tty`意思终端。
  - -v：目录映射，宿主机目录：容器目录，完整为 `--volume`意思是卷。
  - -d：守护进程，后台运行该容器。完整为 `--detach`意思是分离。
  - -p：宿主机端口：容器主机端口映射。完整为 `--publish`,原注释：Publish a container's port(s) to the host（将容器的端口发布到主机）。
  - -u：以什么用户身份创建容器。完整为 `--user`。
  - --name "容器名"：容器名字。
  - -m：设置容器的最大内存值。完整为 `--memory`。
  - -h：指定容器的 host name。完整为 `--hostname`。
  - --dns 8.8.8.8：指定容器 dns 服务器。
  - --privileged：容器内是否使用真正的 root 权限。
  - -e：设置容器内的环境变量。完整为 `--env`。
  - --link：建立一个与指定容器连接的内部网络接口。
  - -w：将参数的路径设置为容器的工作目录。完整为 `--workdir`。

## Dockerfile

> 命令全部大写。
> 使用 `docker build -t 标签 .`

- AGE：创建过程中使用的变量。
  - 格式为 ARG<name>[=<default value>]。
  - 在执行 docker build 时，可以通过-build-arg[=]来为变量赋值。当镜像编译成功后，ARG 指定的变量将不再存在（ENV 指定的变量将在镜像中保留）。
  - Docker 内置了一些镜像创建变量，用户可以直接使用而无须声明，包括（不区分大小写）HTTP_PROXY、HTTPS_PROXY、FTP_PROXY、NO_PROXY。
- FROM：指定创建过程中使用的基础镜像。
  - 格式为：
    - FROM<image>[AS<name>]
    - FROM<image>：<tag>[AS<name>]
    - FROM<image>@<digest>[AS<name>]。
  - 为了保证镜像精简，可以选用体积较小的镜像如 Alpine 或 Debian 作为基础镜像。
- LABEL：为生成的镜像添加元数据标签信息。
  - 格式为 LABEL<key>=<value><key>=<value><key>=<value>...。
- EXPOSE：声明镜像内服务监听的端口。
  - 注意该指令只是起到声明作用，并不会自动完成端口映射。
  - 如果要映射端口出来，在启动容器时可以使用-P 参数（Docker 主机会自动分配一个宿主机的临时端口）或-p HOST_PORT：CONTAINER_PORT 参数（具体指定所映射的本地端口）。
- ENV：指定环境变量，在镜像生成过程中会被后续 RUN 指令使用，在镜像启动的容器中也会存在。
  - 格式为：
    - ENV<key><value>
    - ENV<key>=<value>...。
- ENTRYPOINT：指定镜像默认的入口命令，该入口命令会在启动容器时作为根命令执行，所有传入值作为该命令的参数
  - 支持两种格式：
    - ENTRYPOINT["executable"，"param1"，"param2"]：exec 调用执行；
    - ENTRYPOINT command param1 param2：shell 中执行。
  - 此时，CMD 指令指定值将作为根命令的参数。
  - 每个 Dockerfile 中只能有一个 ENTRYPOINT，当指定多个时，只有最后一个起效。
  - 在运行时，可以被--entrypoint 参数覆盖掉，如 docker run--entrypoint。
- VOLUME：创建一个数据卷挂载点。
  - 格式为 VOLUME["/data"]。
  - 运行容器时可以从本地主机或其他容器挂载数据卷，一般用来存放数据库和需要保持的数据等。
- USER：指定运行容器时的用户名或 UID。
  - 格式为 USER daemon。
  - 当服务不需要管理员权限时，可以通过该命令指定运行用户，并且可以在 Dockerfile 中创建所需要的用户。
  - 如：`RUN groupadd -r postgres && useradd --no-log-init -r -g postgres postgres`
- WORKDIR：配置工作目录。
  - 为后续的 RUN、CMD、ENTRYPOINT 指令配置工作目录。
  - 格式为 WORKDIR /path/to/workdir。
  - 因此，为了避免出错，推荐 WORKDIR 指令中只使用绝对路径。
- ONBUILD：创建子镜像时指定自动执行的操作指令。

  - 格式为 ONBUILD[INSTRUCTION]。
  -

- STOPSIGNAL：指定退出信号值。
  - 指定所创建镜像启动的容器接收退出的信号值
  - STOPSIGNAL signal
- HEALTCHECK：配置容器如何进行健康检查。
  - HEALTHCHECK[OPTIONS]CMD command：根据所执行命令返回值是否为 0 来判断；
  - HEALTHCHECK NONE：禁止基础镜像中的健康检查。
  - OPTION 支持如下参数：
    - -interval=DURATION（default：30s）：过多久检查一次；
    - -timeout=DURATION（default：30s）：每次检查等待结果的超时；
    - -retries=N（default：3）：如果失败了，重试几次才最终确定失败。
- SHELL：指定默认 shell 类型。
  - 指定其他命令使用 shell 时的默认 shell 类型。
  - SHELL ["executable", "parameters"]
- RUN：运行指令命令。
  - 格式为 RUN<command>或 RUN["executable"，"param1"，"param2"]。注意后者指令会被解析为 JSON 数组，因此必须用双引号。前者默认将在 shell 终端中运行命令，即/bin/sh-c；后者则使用 exec 执行，不会启动 shell 环境。
  - 指定使用其他终端类型可以通过第二种方式实现，例如 RUN["/bin/bash"，"-c"，"echo hello"]。
  - 每条 RUN 指令将在当前镜像基础上执行指定命令，并提交为新的镜像层。当命令较长时可以使用\来换行。例如

```dockerfile
RUN apt-get update \
    && apt-get install -y libsnappy-dev zlib1g-dev libbz2-dev \
		&& rm -rf /var/cache/apt \
		&& rm -rf /var/lib/apt/lists/*
```

- CMD：启动容器时默认执行命令。
  - CMD["executable"，"param1"，"param2"]：相当于执行 executable param1 param2，推荐方式；
  - CMD command param1 param2：在默认的 Shell 中执行，提供给需要交互的应用；
  - CMD["param1"，"param2"]：提供给 ENTRYPOINT 的默认参数。
  - 每个 Dockerfile 只能有一条 CMD 命令。如果指定了多条命令，只有最后一条会被执行。
- ADD：添加内容到镜像。
  - 每个 Dockerfile 只能有一条 CMD 命令。如果指定了多条命令，只有最后一条会被执行。
  - 格式为 ADD<src><dest>
  - 其中<src>可以是 Dockerfile 所在目录的一个相对路径（文件或目录）；也可以是一个 URL；还可以是一个 tar 文件（自动解压为目录）<dest>可以是镜像内绝对路径，或者相对于工作目录（WORKDIR）的相对路径。
- COPY：复制内容到镜像。
  - 格式为 COPY<宿主机 src><镜像 dest>。
  - 复制本地主机的<src>（为 Dockerfile 所在目录的相对路径，文件或目录）下内容到镜像中的<dest>。目标路径不存在时，会自动创建。
  - 路径同样支持正则格式。
  - COPY 与 ADD 指令功能类似，当使用本地目录为源目录时，推荐使用 COPY。

### PHP Dockerfile

```dockerfile
FROM php:7.4-fpm

# 设置时区
ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

# 更新安装依赖包和PHP核心拓展
RUN apt-get update && apt-get install -y \
    --no-install-recommends libfreetype6-dev libjpeg62-turbo-dev libzip-dev libmcrypt-dev libpng-dev curl \
    && rm -r /var/lib/apt/lists/* \
    && docker-php-ext-configure gd


# 安装 PECL 拓展，安装Redis，swoole
RUN pecl install redis \
    && pecl install zip \
    && pecl install xdebug \
    && pecl install mongodb \
    && pecl install mcrypt \
    && docker-php-ext-enable redis mongodb  mcrypt zip xdebug\
    && docker-php-ext-install -j$(nproc) gd opcache pdo_mysql gettext

# 安装 Composer
ENV COMPOSER_HOME /root/composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
ENV PATH $COMPOSER_HOME/vendor/bin:$PATH

WORKDIR /web
```

## docker-compose

> Compose 定位是“定义和运行多个 Docker 容器的应用。
> 它允许用户通过一个单独的 docker-compose.yml 模板文件（YAML 格式）来定义一组相关联的应用容器为一个服务栈（stack）。

- 任务（task）：一个容器被称为一个任务。任务拥有独一无二的 ID，在同一个服务中的多个任务序号依次递增。
- 服务（service）：某个相同应用镜像的容器副本集合，一个服务可以横向扩展为多个容器实例。
- 服务栈（stack）：由多个服务组成，相互配合完成特定业务，如 Web 应用服务、数据库服务共同构成 Web 服务栈，一般由一个 docker-compose.yml 文件定义。

### 命令

- 创建并运行容器：`docker-compose up -d`
  - -d：后台运行。
- 查看版本：`docker-compose version`
- 停止用 up 命令启动的容器并移除网络：`docker-compose -f yml文件 down`
- 日志：`docker-compose logs`
  - -f：不间断查看。
  - --tail：查看条数。

### 模板

> 默认的模板文件名称为 docker-compose.yml，格式为 YAML 格式，目前最新的版本为 v3。

#### 命令

> 命令小写。

- build：指定 Dockerfile 所在的文件夹路径。如果使用自定义镜像配置这里。可以是绝对路径或相对路径。

```yaml
version: "3"
services:
  app:
    build: /path/to/build
```

- build 指令还可以指定创建镜像的上下文、Dockerfile 路径、标签、Shm 大小、参数和缓存来源等，例如

```yaml
version: "3"
services:
  app:
    build:
      context: /path/to/build/dir
      dockerfile: Dockerfile
      labels:
        version: "2.0"
        released: "true"
      shm_size: "2gb"
      args:
        key: val
        name: myApp
      cache_from:
        - myApp: 1.0
```

- cap_add、cap_drop：指定容器的内核能力分配。

```yaml
cap_add:
  - ALL
cap_drop:
  - NET_ADMIN
```

- command：覆盖容器启动后默认执行的命令，可以为字符串格式或 JSON 数组格式。
  - command: echo "hello world
  - command: ["bash", "-c", "echo", "hello world"]
- configs：在 Docker Swarm 模式下，可以通过 configs 来管理和访问非敏感的配置信息。支持从文件读取或外部读取。
- cgroup_parent：指定父 cgroup 组，意味着将继承该组的资源限制。
- container_name：指定容器名称。
- devices：指定设备映射关系。
- depends_on：指定多个服务之间的依赖关系。
- dns：自定义 DNS 服务。
- dns_search：配置 DNS 搜索域。
- dockerfile：指定额外的编译镜像的 Dockerfile 文件。
- entrypoint：覆盖容器中默认的入口命令。
- env_file：从文件中获取环境变量。
- environment：设置环境变量。
- expose：暴露端口。
- extends：基于其他模版文件进行扩展。
- external_links：链接到 docker-compose.yml 外部的容器。
- extra_hosts：指定额外的 host 名称映射信息。
- healthcheck：指定检测应用健康状态机制。
- image：指定镜像名称或镜像 ID。
- isolation：配置容器隔离的机制。
- labels：为容器添加 docker 元数据信息。
- links：链接到其他服务中的容器。
- logging：与日志相关的配置。
- network_mode：设置网络模式。
- networks：所加入的网络。
- ports：暴露的端口信息。
- security_opt：指定容器模版标签（label）机制的默认属性（用户、角色、类型、级别）
- stop_grace_period：指定应用停止时，容器的优雅停止期限。过期后通过 SIGKILL 强制退出。默认为 10s
- stop_signal：指定停止的信号。
- sysctls：配置容器内的内核参数。
- ulimits：指定容器的 ulimits 限制。
- userns_mode：指定用户命名空间模式。
- volumes：数据卷所挂载路径设置。
- restart：指定重启策略。
- deploy：指定部署和运行时容器先关配置。

#### 例子

```yaml
version: "3.0"
networks:
  lnmp-net:
    driver: bridge
services:
  swoole_php:
    # container_name: swoole_php
    container_name: my_php
    # image: swoole_php
    build:
      context: ./php
      dockerfile: Dockerfile
      labels:
        - version="2.0"
    ports:
      - 9001:9000
      # - 9003:9003
    volumes:
      - ./www:/web/dockerProject/www/
      - ./php/xdebug:/web/xdebug/
      - ./php/conf.d/docker-php-ext-xdebug.ini /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
    #      - ./php/conf.d/docker-php-ext-xdebug.ini:/usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
    environment:
      XDEBUG_CONFIG: xdebug.client_host=host.docker.internal
    restart: always
    networks:
      - lnmp-net
  nginx:
    container_name: nginx
    image: nginx:1.18.0-alpine
    ports:
      - 80:80
    volumes:
      - ./www:/web/dockerProject/www/
      - ./nginx/web/conf.d:/etc/nginx/conf.d
      - ./nginx/web/logs/web:/var/log/nginx
    networks:
      - lnmp-net
    # links:
    #   -
    # depends_on:
    #   - my_php
```
