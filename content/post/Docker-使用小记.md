---
title: "Docker 使用小记"
date: 2023-03-15 10:07:56
categories: ["技术总结"]
tags: ["Docker"]
url: docker

################################目录################################
toc: true
autoCollapseToc: false
################################公式渲染################################
mathjax: false

################################基本不动################################
# lastmod: {{ .Date }}
draft: false
# keywords: []
# description: ""
# tags: []
author: "Yaxing"

comment: false
postMetaInFooter: false
hiddenFromHomePage: false
contentCopyright: true
---

本文对 Docker 的配置与使用做简单小记。<!--more-->

## 镜像

### 列出镜像列表

```sh
docker image ls
docker image ls ubuntu
docker image ls ubuntu:18.04
```

### 查找镜像

```sh
docker search redis
```

### 获取镜像

```sh
docker pull ubuntu:18.04
```

### 删除本地镜像

```sh
docker image rm <IMAGE ID>
```

## 容器

### 启动容器

```sh
docker run -it ubuntu bash
```

参数说明

- **-i**: 交互式操作。
- **-t**: 终端。
- **ubuntu**: ubuntu 镜像。
- **bash**：放在镜像名后的是命令，这里我们希望有个交互式 Shell，因此用的是 bash。

### 启动已停止运行的容器

查看所有的容器命令如下：

```
docker ps -a
```

使用 docker start 启动一个已停止的容器：

```
docker start <容器 ID> 
```

### 后台运行

在大部分的场景下，我们希望 docker 的服务是在后台运行的，我们可以过 **-d** 指定容器的运行模式。

```
$ docker run -itd --name ubuntu-test ubuntu bash
```

### 停止一个容器

```
$ docker stop <容器 ID>
```

停止的容器可以通过 docker restart 重启：

```
$ docker restart <容器 ID>
```

### 进入容器

在使用 **-d** 参数时，容器启动后会进入后台。此时想要进入容器，可以通过以下指令进入：

```
docker exec -it <容器 ID> bash
```

### 删除容器

```
$ docker rm <容器 ID>
```

## 实例

### MySQL

```sh
docker pull mysql:8.0.32
# 启动
docker run -d --name mysql -e MYSQL_ROOT_PASSWORD=FYX123fyx -p 3306:3306 mysql:8.0.32 
# 进入容器
docker exec -it mysql bash

# 登录mysql
mysql -uroot -p
ALTER user 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'FYX123fyx';
FLUSH PRIVILEGES; 
```

### Redis

```sh
docker run -d --name redis -p 6379:6379 redis
docker exec -it redis bash
```

### Nginx

```sh
docker run -d --name nginx -p 80:80 nginx
docker exec -it nginx bash
```

### Tomcat

```sh
docker pull tomcat:8
docker run -d --name tomcat -p 8080:8080 tomcat:8
docker exec -it tomcat bash
```

访问 localhost:8080显示404，是因为没有资源可以显示

```sh
rm -rf webapps
mv webapps.dist/ webapps
```

重新访问，正常显示

为了保存当前状态，以后可以正常访问，要提交commit新建镜像

```sh
docker commit --author "Yaxing Fang <yaxingfang@gmail.com>" --message "修改tomcat主页显示" tomcat tomcat:v1
# 通过该镜像创建容器启动
docker run -d --name tomcat -p 8080:8080 tomcat:v1
```

## 参考

[Docker — 从入门到实践](https://yeasy.gitbook.io/docker_practice/)
