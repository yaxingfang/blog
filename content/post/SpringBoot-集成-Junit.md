---
title: "SpringBoot 集成 Junit"
date: 2023-02-14 09:26:51
categories: ["技术总结"]
tags: ["单元测试", "SpringBoot"]
url: springboot-junit

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

本文对 SpringBoot 集成 Junit 做简单小记。<!--more-->

1、添加依赖

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-test</artifactId>
  <scope>test</scope>
</dependency>
<dependency>
  <groupId>junit</groupId>
  <artifactId>junit</artifactId>
  <scope>test</scope>
</dependency>
```

2、编写测试类

```java
@RunWith(SpringRunner.class)
@SpringBootTest(classes = SpringBootDemoApplication.class)
class SpringBootDemoApplicationTests {
    @Autowired
    private ScoreDao scoreDao;

    @Test
    void queryAllScoresTest() {
        List<Score> scores = scoreDao.queryAllScores();
        System.out.println(scores);
    }
}
```

3、测试运行

```
[Score(id=1, name=小明, subject=语文, score=92, create_time=Mon Feb 13 19:49:00 CST 2023, update_time=Mon Feb 13 20:03:48 CST 2023)]
```
