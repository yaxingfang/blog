---
title: Mac 安装 MySQL 以及 Sequel Pro 配置与连接
copyright: true
mathjax: false
categories:
  - Mac开发环境配置
toc: false
date: 2023-02-11 10:23:54
tags:
urlname: mysql-sequel-pro
---

本文介绍在 Mac 上安装 MySQL 以及使用 Sequel Pro 作为可视化客户端的配置与连接。<!--more-->

1、安装 MySQL

可以在官网下载 .dmg 文件进行安装也可以使用 homebrew 安装 MySQL，以下使用 homebrew 安装。

```sh
brew install mysql
```

注意：设置密码的时候设置的不要太简单，要包含字母、数字、字符（太简单的话后续出问题还要再重新改成一个复杂的）

2、下载 Sequel Pro 

Sequel Pro下载地址：http://www.sequelpro.com/

下载安装后，出现连接界面，填入信息，此时可能报错 Connection failed MySQL said: Authentication plug ... cannot be loaded ...

解决方法如下：

用终端连接MySQL，然后执行以下命令：

```bash
mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '此处填写设置的MySQL密码';
```

如果报错显示密码简单，则通过以下方式修改密码：

```sh
mysqladmin -u用户名 -p旧密码 password 新密码 
```

然后重新执行第一条命令，Sequel Pro 可以正常连接。

P.S. 本机 Sequel Pro 导入 SQL 建表语句 loading 过久，不好用，不如使用 IDEA 自带的数据库工具，或者可以使用 Jet 家的 DataGrip。
