---
title: "SpringBoot 集成 Swagger"
date: 2023-02-14 09:26:39
categories: ["技术总结"]
tags: ["Swagger", "SpringBoot"]
url: springboot/swagger

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

本文对 SpringBoot 集成 Swagger 配置和使用做简单小记。<!--more-->

## 配置

1. 引入依赖

	```xml
	<dependency>
	    <groupId>io.springfox</groupId>
	    <artifactId>springfox-swagger2</artifactId>
	    <version>2.9.2</version>
	</dependency>
	
	<dependency>
	    <groupId>io.springfox</groupId>
	    <artifactId>springfox-swagger-ui</artifactId>
	    <version>2.9.2</version>
	</dependency>
	```

2. 创建 Swagger 2 的配置类

	```java
	@Configuration
	@EnableSwagger2
	public class SwaggerConfig {
	    /* swagger会帮助我们⽣成接⼝⽂档
	     * 1：配置⽣成的⽂档信息
	     * 2: 配置⽣成规则
	     */
	    /* Docket封装接⼝⽂档信息 */
	    @Bean
	    public Docket getDocket() {
	
	        //创建封⾯信息对象
	        ApiInfoBuilder apiInfoBuilder = new ApiInfoBuilder();
	        apiInfoBuilder
	                .title("后端接⼝说明")
	                .description("此⽂档详细说明了SpringBootDemo项⽬后端接⼝规范")
	                .version("1.0")
	                .contact(new Contact("yaxing", "https://yaxing97.com", "yaxingfang@163.com"));
	        ApiInfo apiInfo = apiInfoBuilder.build();
	        Docket docket = new Docket(DocumentationType.SWAGGER_2)
	                .apiInfo(apiInfo)
	                .select()
	                .apis(RequestHandlerSelectors.basePackage("com.yaxing.controller"))
	                .paths(PathSelectors.any())
	                .build();
	        return docket;
	    }
	}
	```

3. 运行

	运行主启动类，此时若报错`Failed to start bean 'documentationPluginsBootstrapper'`，解决方案参考https://blog.csdn.net/mapboo/article/details/121568519，降低springboot的版本或添加以下配置。

	```yaml
	spring:
	  mvc:
	    pathmatch:
	      matching-strategy: ANT_PATH_MATCHER
	```

	访问 http://localhost:8080/swagger-ui.html

## 注解使用

1. `@Api` 类注解，在控制器类上，可以对控制器类进行说明

	```java
	@Api(tags = "分数管理", value = "提供分数管理相关接口")
	public class ScoreController {}
	```

2. `@ApiOperation` 方法注解，在接口方法上，可以对该接口方法进行说明

	```java
	@ApiOperation("新增用户学科分数")
	public String addScore(String name, String subject, Integer score) {}
	```

	显示：GET /user/login 用户登陆接口

3. `@ApiImplicitParams`和`@ApiImplicitParam`对接口的方法参数进行说明

	```java
	@ApiOperation("新增用户学科分数")
	@ApiImplicitParams({
	  @ApiImplicitParam(name = "name", value = "姓名", paramType = "String"),
	  @ApiImplicitParam(name = "subject", value = "学科", paramType = "String"),
	  @ApiImplicitParam(name = "score", value = "分数", paramType = "Integer")
	})
	public String addScore(String name, String subject, Integer score) {}
	```

4. `@ApiModel`和`@ApiModelProperty` 对实体类添加注解说明

	```java
	@ApiModel
	public class Score {
	    /**
	     * 主键
	     */
	    @ApiModelProperty(value = "主键", dataType = "Long")
	    private Long id;
	}
	```

5. `@ApiIgnore`方法注解，如果不想对某个接口方法进行说明，在接口方法上添加该注解

## UI 插件

1. 将 springfox-swagger-ui 修改为以下

	```xml
	<dependency>
	    <groupId>com.github.xiaoymin</groupId>
	    <artifactId>swagger-bootstrap-ui</artifactId>
	    <version>1.9.6</version>
	</dependency>
	```

2. 启动主启动类，打开 http://localhost:8080/doc.html
