---
title: "Logback + Slf4j 使用小记"
date: 2023-02-13 15:09:20
categories: ["技术总结"]
tags: ["日志"]
url: Logback-Slf4j

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

本文对 Logback + Slf4j 的配置和使用做简单小记。<!--more-->

## 简介

Logback + Slf4j 是主流的日志框架 + 日志门面的使用组合。

## 配置

1、导入依赖

```xml
<!-- https://mvnrepository.com/artifact/ch.qos.logback/logback-classic -->
<dependency>
  <groupId>ch.qos.logback</groupId>
  <artifactId>logback-classic</artifactId>
  <version>1.2.3</version>
</dependency>
<dependency>
  <groupId>org.projectlombok</groupId>
  <artifactId>lombok</artifactId>
  <version>1.18.22</version>
</dependency>

<dependency>
  <groupId>junit</groupId>
  <artifactId>junit</artifactId>
  <version>4.12</version>
  <scope>test</scope>
</dependency>
```

2、配置 `logback.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- scan属性未true时，如果配置文档发生改变将会进行重新加载 -->
<!-- scanPeriod属性设置监测配置文件修改的时间间隔，默认单位为毫秒，在scan为true时才生效 -->
<!-- debug属性如果为true时，会打印出logback内部的日志信息 -->
<configuration scan="true" scanPeriod="60 seconds" debug="false">
    <!-- 定义参数常量 -->
    <!-- 日志级别：TRACE<DEBUG<INFO<WARN<ERROR，其中常用的是DEBUG、INFO和ERROR -->
    <property name="log.level" value="debug" />
    <!-- 文件保留时间 -->
    <property name="log.maxHistory" value="30" />
    <!-- 日志存储路径 -->
    <property name="log.filePath" value="./logs/" />
    <!-- 日志的显示格式 -->
    <property name="log.pattern"
              value="%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50}-%msg%n"/>
    <!-- 用于说明输出介质，此处说明控制台输出 -->
    <appender name="consoleAppender"
              class="ch.qos.logback.core.ConsoleAppender">
        <!-- 类似于layout，除了将时间转化为数组，还会将转换后的数组输出到相应的文件中 -->
        <encoder>
            <!-- 定义日志的输出格式 -->
            <pattern>${log.pattern}</pattern>
        </encoder>
    </appender>
    <!-- 所有日志，表示文件随着时间的推移按时间生成日志文件 -->
    <appender name="allLogAppender"
              class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 文件路径 -->
        <file>${log.filePath}/allLog.log</file>
        <!-- 滚动策略 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- 设置文件名称 -->
            <fileNamePattern>
                ${log.filePath}/all/allLog.%d{yyyy-MM-dd}.log
            </fileNamePattern>
            <!-- 设置最大保存周期 -->
            <MaxHistory>${log.maxHistory}</MaxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>${log.pattern}</pattern>
        </encoder>
        <!-- &lt;!&ndash; 过滤器，过滤掉不是指定日志水平的日志 &ndash;&gt; -->
        <!-- <filter class="ch.qos.logback.classic.filter.LevelFilter"> -->
        <!--     &lt;!&ndash; 设置日志级别 &ndash;&gt; -->
        <!--     <level>DEBUG</level> -->
        <!--     &lt;!&ndash; 如果跟该日志水平相匹配，则接受 &ndash;&gt; -->
        <!--     <onMatch>ACCEPT</onMatch> -->
        <!--     &lt;!&ndash; 如果跟该日志水平不匹配，则过滤掉 &ndash;&gt; -->
        <!--     <onMismatch>DENY</onMismatch> -->
        <!-- </filter> -->
    </appender>
    <!-- DEBUG，表示文件随着时间的推移按时间生成日志文件 -->
    <appender name="debugAppender"
              class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 文件路径 -->
        <file>${log.filePath}/debug.log</file>
        <!-- 滚动策略 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- 设置文件名称 -->
            <fileNamePattern>
                ${log.filePath}/debug/debug.%d{yyyy-MM-dd}.log
            </fileNamePattern>
            <!-- 设置最大保存周期 -->
            <MaxHistory>${log.maxHistory}</MaxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>${log.pattern}</pattern>
        </encoder>
        <!-- 过滤器，过滤掉不是指定日志水平的日志 -->
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <!-- 设置日志级别 -->
            <level>DEBUG</level>
            <!-- 如果跟该日志水平相匹配，则接受 -->
            <onMatch>ACCEPT</onMatch>
            <!-- 如果跟该日志水平不匹配，则过滤掉 -->
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>
    <!-- INFO，表示文件随着时间的推移按时间生成日志文件 -->
    <appender name="infoAppender"
              class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 文件路径 -->
        <file>${log.filePath}/info.log</file>
        <!-- 滚动策略 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- 设置文件名称 -->
            <fileNamePattern>
                ${log.filePath}/info/info.%d{yyyy-MM-dd}.log
            </fileNamePattern>
            <!-- 设置最大保存周期 -->
            <MaxHistory>${log.maxHistory}</MaxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>${log.pattern}</pattern>
        </encoder>
        <!-- 过滤器，过滤掉不是指定日志水平的日志 -->
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <!-- 设置日志级别 -->
            <level>INFO</level>
            <!-- 如果跟该日志水平相匹配，则接受 -->
            <onMatch>ACCEPT</onMatch>
            <!-- 如果跟该日志水平不匹配，则过滤掉 -->
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>
    <!-- WARN，表示文件随着时间的推移按时间生成日志文件 -->
    <appender name="warnAppender"
              class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 文件路径 -->
        <file>${log.filePath}/warn.log</file>
        <!-- 滚动策略 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- 设置文件名称 -->
            <fileNamePattern>
                ${log.filePath}/warn/warn.%d{yyyy-MM-dd}.log
            </fileNamePattern>
            <!-- 设置最大保存周期 -->
            <MaxHistory>${log.maxHistory}</MaxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>${log.pattern}</pattern>
        </encoder>
        <!-- 过滤器，过滤掉不是指定日志水平的日志 -->
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <!-- 设置日志级别 -->
            <level>WARN</level>
            <!-- 如果跟该日志水平相匹配，则接受 -->
            <onMatch>ACCEPT</onMatch>
            <!-- 如果跟该日志水平不匹配，则过滤掉 -->
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>
    <!-- ERROR，表示文件随着时间的推移按时间生成日志文件 -->
    <appender name="errorAppender"
              class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 文件路径 -->
        <file>${log.filePath}/error.log</file>
        <!-- 滚动策略 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- 设置文件名称 -->
            <fileNamePattern>
                ${log.filePath}/error/error.%d{yyyy-MM-dd}.log
            </fileNamePattern>
            <!-- 设置最大保存周期 -->
            <MaxHistory>${log.maxHistory}</MaxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>${log.pattern}</pattern>
        </encoder>
        <!-- 过滤器，过滤掉不是指定日志水平的日志 -->
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <!-- 设置日志级别 -->
            <level>ERROR</level>
            <!-- 如果跟该日志水平相匹配，则接受 -->
            <onMatch>ACCEPT</onMatch>
            <!-- 如果跟该日志水平不匹配，则过滤掉 -->
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <!-- 用于存放日志对象，同时指定关联的package位置 -->
    <!-- name指定关联的package -->
    <!-- level表明指记录哪个日志级别以上的日志 -->
    <!-- appender-ref指定logger向哪个文件输出日志信息 -->
    <!-- additivity为true时，logger会把根logger的日志输出地址加入进来，但logger水平不依赖于根logger -->
    <!-- <logger name="com.campus.o2o" level="${log.level}" additivity="true"> -->
    <!--     <appender-ref ref="debugAppender" /> -->
    <!--     <appender-ref ref="infoAppender" /> -->
    <!--     <appender-ref ref="errorAppender" /> -->
    <!-- </logger> -->

    <!-- 特殊的logger，根logger -->
    <root level="info">
        <!-- 指定默认的日志输出 -->
        <appender-ref ref="consoleAppender" />
        <appender-ref ref="allLogAppender" />
        <appender-ref ref="debugAppender" />
        <appender-ref ref="infoAppender" />
        <appender-ref ref="warnAppender" />
        <appender-ref ref="errorAppender" />
    </root>
</configuration>
```

## 使用

```java
@Slf4j
public class LogbackTest {
    @Test
    public void test() {
        log.trace("=====trace=====");
        log.debug("=====debug=====");
        log.info("=====info=====");
        log.warn("=====warn=====");
        log.error("=====error=====");
    }
}
```

```
2023-02-13 15:07:36.226 [main] INFO  com.yaxing.LogbackTest-=====info=====
2023-02-13 15:07:36.228 [main] WARN  com.yaxing.LogbackTest-=====warn=====
2023-02-13 15:07:36.229 [main] ERROR com.yaxing.LogbackTest-=====error=====
```

## 参考

https://www.cnblogs.com/xrq730/p/8628945.html

https://www.cnblogs.com/warking/p/5710303.html
