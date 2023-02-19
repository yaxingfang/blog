---
title: Maven 聚合工程
copyright: true
mathjax: false
categories:
  - 技术总结
  - Maven
toc: false
date: 2023-02-19 10:21:34
tags:
urlname: maven-aggregation-project
---

本文对 Maven 聚合工程的搭建做简单小记。<!--more-->

Maven 聚合工程就是可以在一个 Maven 父工程中创建多个组件（项目），这个多个组件之间可以相互依赖，实现组件的复用。

## 父工程

新建父工程 CommonDemo（Maven 工程）

修改父工程的打包方式

```xml
<packaging>pom</packaging>
```

在父工程中定义依赖版本以及版本依赖管理（定义版本依赖管理 dependencyManagement 不会引入依赖）

```xml
<properties>
    <maven.compiler.source>8</maven.compiler.source>
    <maven.compiler.target>8</maven.compiler.target>
    <springboot.version>2.7.8</springboot.version>
    <junit.version>4.12</junit.version>
    <druid.version>1.2.8</druid.version>
    <mysql-connector.version>8.0.28</mysql-connector.version>
    <mybatis.version>2.3.0</mybatis.version>
    <lombok.version>1.18.24</lombok.version>
    <logback.version>1.2.3</logback.version>
    <redis.version>2.7.8</redis.version>
    <aop.version>2.7.8</aop.version>
</properties>

<dependencyManagement>
    <dependencies>
        <!-- web -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <version>${springboot.version}</version>
        </dependency>

        <!-- 测试 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <version>${springboot.version}</version>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>${junit.version}</version>
            <scope>test</scope>
        </dependency>

        <!-- database -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
            <version>${springboot.version}</version>
        </dependency>

      	<!-- 德鲁伊连接池 -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
            <version>${druid.version}</version>
        </dependency>

      	<!-- mysql 连接驱动 -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>${mysql-connector.version}</version>
        </dependency>

        <!-- redis -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
            <version>${redis.version}</version>
        </dependency>

        <!-- mybatis -->
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>${mybatis.version}</version>
        </dependency>

        <!-- lombok -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>${lombok.version}</version>
            <scope>provided</scope>
        </dependency>

        <!-- swagger -->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
            <version>2.9.2</version>
        </dependency>
        <dependency>
            <groupId>com.github.xiaoymin</groupId>
            <artifactId>swagger-bootstrap-ui</artifactId>
            <version>1.9.6</version>
        </dependency>

        <!-- 日志 -->
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
            <version>${logback.version}</version>
        </dependency>

        <!-- aop -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-aop</artifactId>
            <version>${aop.version}</version>
        </dependency>

        <!-- 热部署 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <version>${springboot.version}</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```

## 子工程

在 CommonDemo 下新建 Module

通过定义子工程的 \<dependencies\> 标签即可引入相应依赖（版本号继承自父工程的版本号）

```xml
<dependencies>
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
  </dependency>

  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
  </dependency>

  <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
  </dependency>
<dependencies>
```
