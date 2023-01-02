---
title: PHP Xdebug
urlname: chz6uvhacgtg69x1
date: '2022-12-28 20:07:41 +0800'
tags: []
categories: []
---

tags: [xdebug]
categories: [PHP]
cover: ''

---

## 安装

```bash
pecl install xdebug-3.0.4
#注意 xdebug2 和 xdebug3 配置改版

#查看 php 配置文件
php --ini
#修改 php.ini 并配置
#xdebug3 将 extension 改为 zend_extension
zend_extension="/usr/local/opt/php@7.4/pecl/20190902/xdebug.so"
xdebug.idekey="PHPSTORM"  # 非常重要，务必记住
;配置端口和监听的域名
xdebug.mode=debug
xdebug.discover_client_host=true
xdebug.remote_cookie_expire_time = 3600
xdebug.client_port=9000
xdebug.client_host="localhost"
xdebug.start_with_request=yes
xdebug.remote_handler = "dbgp"
```

## docker 中调试

> 关键就是要获取宿主机 ip。

### docker-compose.yml

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

### dockerfile

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
    && pecl install xdebug-3.0.4\
    && pecl install mongodb \
    && pecl install mcrypt \
    && docker-php-ext-enable redis mongodb  mcrypt zip xdebug\
    && docker-php-ext-install -j$(nproc) gd opcache pdo_mysql gettext

# 安装 Composer
ENV COMPOSER_HOME /root/composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
ENV PATH $COMPOSER_HOME/vendor/bin:$PATH
RUN composer config -g secure-http false \
    && composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/
#将本地 xdebug 复制到镜像中
COPY ./conf.d/docker-php-ext-xdebug.ini /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini

WORKDIR /web
```

### launch.json

```json
{
  // 使用 IntelliSense 了解相关属性。
  // 悬停以查看现有属性的描述。
  // 欲了解更多信息，请访问: https://go.microsoft.com/fwlink/?linkid=830387
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Listen for Xdebug",
      "type": "php",
      "request": "launch",
      "port": 9003,
      "pathMappings": {
        //这里配置 docker 和宿主机文件映射
        "/web/dockerProject/www/conghua-server": "${workspaceFolder}"
      },
      "log": true
    },
    {
      "name": "Launch currently open script",
      "type": "php",
      "request": "launch",
      "program": "${file}",
      "cwd": "${fileDirname}",
      "port": 9003
    }
  ]
}
```

### docker-php-ext-xdebug.ini

```properties

zend_extension=xdebug
xdebug.idekey="PHPSTORM"
; 非常重要，务必记住
;配置端口和监听的域名
xdebug.mode=debug
xdebug.discover_client_host=true
xdebug.remote_cookie_expire_time = 3600
xdebug.client_port=9003
xdebug.client_host="192.168.1.6"
; xdebug.client_host=host.docker.internal
xdebug.start_with_request=yes
xdebug.remote_handler = "dbgp"
xdebug.log=/web/xdebug/xdebug.log
```
