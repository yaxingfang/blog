---
title: "SpringBoot 热部署配置"
date: 2023-02-14 09:26:06
categories: ["技术总结"]
tags: ["SpringBoot", "热部署"]
url: spring-boot-hot-build

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

本文对 SpringBoot 热部署配置做简单小记。<!--more-->

项目首次部署，服务启动之后，如果应用发生了变化，让 IDEA 感知到应用的变化，自动完成 jar 的更新，无需手动再次启动服务器，就可以访问应用的更新。

1. Settings > Build > Compiler，勾选 Build project automatically

2. 双击 shift 搜索 Registry，进入后勾选 Compile autoMake allow when app running

	如果没有该选项，则 Settings > Advanced Settings，勾选 Allow auto-make to start even if developed application is currently running（在运行时也会自动编译，可以不勾选该项）

3. 配置项目 on update action 和 on frame deactivation 为更新 class 和 resources

4. 导入spring-boot-devtools坐标

	```xml
	<dependency>
	    <groupId>org.springframework.boot</groupId>
	    <artifactId>spring-boot-devtools</artifactId>
	</dependency>
	```

5. 在 application.yml 文件中开启热部署配置

	```yaml
	spring:
	  devtools:
	    remote:
	      restart:
	        enabled: true
	```
