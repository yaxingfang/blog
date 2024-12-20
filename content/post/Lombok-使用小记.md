---
title: "Lombok 使用小记"
date: 2023-02-13 15:09:10
categories: ["技术总结"]
tags: ["Lombok"]
url: lombok

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

本文对 Lombok 的使用做简单小记。<!--more-->

## Lombok 简介

Lombok 利用注解自动生成 Java Bean 中的 Getter、Setter、Equals、ToString、HashCode 等方法，省去自己添加这些方法。

## Lombok 安装

1、IDEA 设置

​	1.1、安装插件 Lombok

​	1.2、在设置 Build > Compiler > Annotation Processors 中，勾选 Enable annotation processing

2、引入依赖

```xml
<!-- https://mvnrepository.com/artifact/org.projectlombok/lombok -->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.24</version>
    <scope>provided</scope>
</dependency>
```

## Lombok 使用

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Person {
    private String name;
    private String address;
    private Integer age;
    private String hobbit;
    private String phone;
}
```

`@Data`：注在类上，提供类的 Getter、Setter、Equals、ToString、HashCode 方法
`@NoArgsConstructor`：注在类上，提供类的无参构造
`@AllArgsConstructor`：注在类上，提供类的全参构造
