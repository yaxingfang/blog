---
title: 从 0 搭建 SpringBoot 项目
copyright: true
mathjax: false
categories:
  - 技术总结
  - SpringBoot
toc: false
date: 2023-02-14 09:24:21
tags:
urlname: springboot-stater
---

本文对从 0 搭建一个 SpringBoot 项目并实现简单查询逻辑进行小记。<!--more-->

## 建表

```mysql
create table score
(
    id          bigint unsigned auto_increment comment '主键'
        primary key,
    name        varchar(255)                       not null comment '姓名',
    subject     varchar(255)                       not null comment '科目',
    score       int                                not null comment '得分',
    create_time datetime default CURRENT_TIMESTAMP not null comment '创建时间',
    update_time datetime default CURRENT_TIMESTAMP not null on update CURRENT_TIMESTAMP comment '更新时间'
);
```

## 新建项目

新建SpringBoot项目，勾选依赖 Lombok、Spring Web、MyBatis Framework

手动添加依赖 mysql-connector-java、spring-boot-starter-test、spring-boot-starter-jdbc、druid-spring-boot-starter

## 修改配置文件

修改 application.properties 后缀为 yaml，配置数据库连接池和 mybatis 映射路径

```yaml
spring:
  datasource:
    druid:
      driver-class-name: com.mysql.cj.jdbc.Driver
      url: jdbc:mysql://localhost:3306/java?characterEncoding=utf8&useSSL=false&serverTimezone=Asia/Shanghai
      username: root
      password: FYX123fyx.
mybatis:
  mapper-locations: classpath:mappers/*Mapper.xml
  type-aliases-package: com.yaxing.entity
```

在 IDEA 的数据库连接中连接数据库，右键相应的表，使用 MyBatisX 生成实体类和 mapper 文件

实体类如下

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Score implements Serializable {
    /**
     * 主键
     */
    private Long id;

    /**
     * 姓名
     */
    private String name;

    /**
     * 科目
     */
    private String subject;

    /**
     * 得分
     */
    private Integer score;

    /**
     * 创建时间
     */
    private Date create_time;

    /**
     * 更新时间
     */
    private Date update_time;
}
```

mapper生成好之后在 SpringBoot 启动类上加注解 `@MapperScan("com.yaxing.dao")`，并设置头文件中对应dao的namespace

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.yaxing.dao.ScoreDao">

    <resultMap id="BaseResultMap" type="com.yaxing.entity.Score">
            <id property="id" column="id" jdbcType="BIGINT"/>
            <result property="name" column="name" jdbcType="VARCHAR"/>
            <result property="subject" column="subject" jdbcType="VARCHAR"/>
            <result property="score" column="score" jdbcType="INTEGER"/>
            <result property="create_time" column="create_time" jdbcType="TIMESTAMP"/>
            <result property="update_time" column="update_time" jdbcType="TIMESTAMP"/>
    </resultMap>

    <sql id="Base_Column_List">
        id,name,subject,
        score,create_time,update_time
    </sql>

    <select id="queryAllScores" resultType="com.yaxing.entity.Score">
        select <include refid="Base_Column_List"/> from score
    </select>
</mapper>
```

得到 dao 如下

```java
@Mapper
public interface ScoreDao {
    List<Score> queryAllScores();
}
```

## 业务逻辑实现

接下来编写 service 和 controller

service

```java
public interface ScoreService {
    List<Score> queryAllScores();
}
```

```java
@Service
public class ScoreServiceImpl implements ScoreService {

    @Autowired
    private ScoreDao scoreDao;

    @Override
    public List<Score> queryAllScores() {
        return scoreDao.queryAllScores();
    }
}
```

controller

```java
@RestController
@RequestMapping("/score")
public class ScoreController {

    @Autowired
    private ScoreService scoreService;

    @GetMapping("/queryAll")
    public List<Score> queryAllScores() {
        return scoreService.queryAllScores();
    }
}
```
