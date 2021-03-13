---
title: docker 搭 nginx、mysql、redis 服务
date: 2021-03-13 17:21:23
tags: [docker,nginx,mysql,redis]
categories: docker
---

# 背景

最近重构项目快完成了，快要上线，用 docker 来搭一些项目依赖的服务，nginx、mysql、redis，下面记录一下步骤

# nginx

## 拉取 nginx 镜像

```shell
docker pull nginx
```

## 编写 docker-compose.yml 文件

```shell
version: '3'

services:
  nginx:
    image: nginx
    container_name: nginx
    volumes: ["/mnt/nginx/config.d:/etc/nginx/conf.d", "/mnt/static:/static"]
    ports:
      - "80:80"
```

## conf 文件配置静态文件服务

```shell
server {
        listen 80;
        server_name localhost;
        location /vehicle-service/static/image/ {
                root /static;
                index index.html;
        }
}

```

<!-- more -->

# mysql

## 拉取 mysql 镜像

```shell
docker pull mysql
```

## 编写 docker-compose.yml 文件

```shell
version: '3'
services:
  mysql:
    image: mysql
    container_name: mysql
    ports:
      - "3306:3306"
    volumes:
    - /mnt/mysql/data:/var/lib/mysql
    - /mnt/mysql/initdb:/docker-entrypoint-initdb.d
    - /mnt/mysql/cnf/my.cnf:/etc/my.cnf
    - /mnt/mysql/cnf/mysql:/etc/mysql
    - /mnt/mysql/mysql-files:/var/lib/mysql-files
    command: [
            '--character-set-server=utf8mb4',
            '--collation-server=utf8mb4_unicode_ci',
            '--max_connections=3000'
    ]
    environment:
      MYSQL_ROOT_PASSWORD: "xxxxxxxxxxxxxxx"
      TZ: "Asia/Shanghai"
```

## 配置 mysql root 账户可远程连接

```shell
docker exec -it mysql /bin/bash #进入容器
use mysql;
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'xxxxxxxxxxxxx';
grant all privileges on *.* to 'root'@'%';
flush privileges;
```

# redis

## 拉取 redis 镜像

```shell
docker pull redis
```

## 编写 docker-compose.yml 文件

```shell
version: '3.5'
services:
  redis:
    image: "redis:5"
    container_name: redis
    command: redis-server /etc/redis/redis.conf
    ports:
      - "6379:6379"
    volumes:
      - /mnt/redis/data:/data
      - /mnt/redis/redis.conf:/etc/redis/redis.conf
```

# 常用命令

```shell
docker-compose up -d   #启动当前目录下 docker-compose.yml 文件的容器
docker-compose down    #移除镜像，如果修改了 docker-compose.yml 需要重新创建容器才生效
docker-compose restart #重启容器，不会删除容器，在修改 conf 文件后，需要 restart 生效 
```

