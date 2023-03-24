---
title: "Maven 使用小记"
date: 2023-02-19 10:18:20
categories: ["技术总结"]
tags: ["Maven"]
url: maven-base

################################目录################################
toc: false
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

本文对 Maven 的安装、配置与基本使用做简单小记。<!--more-->

## Maven 安装

首先从 Maven 官方地址：http://maven.apache.org/download.cgi 下载 apache-maven-xxx-bin.tar.gz

将下载的文件解压到 /Users/yaxing/dev/ 下。

在终端输入：

```sh
vim ~/.bash_profile
```

添加如下内容，配置环境变量：

```sh
export M2_HOME=/Users/yaxing/dev/apache-maven-3.8.6
export PATH=${PATH}:${M2_HOME}/bin
```

让其生效：

```sh
source ~/.bash_profile
```

查看是否安装成功，终端输入：

```sh
mvn -v
```

如果安装成功则显示：

```sh
Apache Maven 3.8.6 (84538c9988a25aec085021c365c560670ad80f63)
Maven home: /Users/yaxing/dev/apache-maven-3.8.6
Java version: 1.8.0_341, vendor: Oracle Corporation, runtime: /Library/Java/JavaVirtualMachines/jdk1.8.0_341.jdk/Contents/Home/jre
Default locale: zh_CN, platform encoding: UTF-8
OS name: "mac os x", version: "12.5.1", arch: "x86_64", family: "mac"
```



## Maven 配置

修改安装目录下的 conf > settings.xml 文件

```sh
vim /Users/yaxing/dev/apache-maven-3.8.6/conf/settings.xml
```

1、配置本地仓库

```xml
<localRepository>/Users/yaxing/dev/apache-maven-3.8.6/mvn-repo</localRepository>
```

2、配置阿里云镜像

```xml
<mirror>
	<id>alimaven</id>
	<mirrorOf>central</mirrorOf>
	<name>aliyun maven</name>
	<url>http://maven.aliyun.com/nexus/content/repositories/central/</url>
</mirror>
```

3、配置全局 jdk 版本

```xml
<!-- java版本 --> 
<profile>
  <id>jdk-1.8</id>
  <activation>
	<activeByDefault>true</activeByDefault>
	<jdk>1.8</jdk>
  </activation>

  <properties>
	<maven.compiler.source>1.8</maven.compiler.source>
	<maven.compiler.target>1.8</maven.compiler.target>
	<maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>
  </properties>
</profile>
```



## Maven 问题

遇到 `zsh: command not found: mvn` 的问题，可参考 https://blog.csdn.net/With_Her/article/details/104144109 解决。



## Maven 使用

### 依赖范围

> 在通过 dependency 添加依赖时，可以通过 `scope` 标签配置当前依赖的适用范围

- test  只在项目测试阶段引入当前依赖(编译、测试)

	```xml
	<dependency>
	    <groupId>junit</groupId>
	    <artifactId>junit</artifactId>
	    <version>4.12</version>
	    <scope>test</scope>
	</dependency>
	```

- runtime 只在运行时使用（运行、测试运行）

- provided 在（编译、测试、运行）

- compile 在（编译、测试、运行、打包）都引入

### 生命周期

| 命令     | 名称     | 描述                                                    |
| -------- | -------- | ------------------------------------------------------- |
| clean    | 清理缓存 | 清理项目生成的缓存                                      |
| validate | 校验     | 验证项目需要是正确的（项目信息、依赖）                  |
| compile  | 编译     | 编译项目专供的源代码                                    |
| test     | 测试     | 运行项目中的单元测试                                    |
| package  | 打包     | 将项目编译后的代码打包成发布格式                        |
| verify   | 检查     | 对集成测试的结果进行检查、确保项目的质量是达标的        |
| install  | 安装     | 将包安装到maven的本地仓库，本地的其他项目可以引用此项目 |
| deploy   | 部署     | 将包安装到私服的仓库，以供其他开发人员共享              |

