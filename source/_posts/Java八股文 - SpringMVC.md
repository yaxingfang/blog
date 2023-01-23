---
title: Java八股文 - SpringMVC
copyright: true
mathjax: false
categories:
  - Java八股文
date: 2023-01-23 16:14:20
tags:
toc: true
urlname: java-springmvc
---

> 整理的SpringMVC相关知识点和面试题，部分内容摘自网络，如有侵权请联系我～<!--more-->

## 概述

### 什么是Spring MVC？简单介绍下你对Spring MVC的理解？

Spring MVC是一个基于Java的实现了MVC设计模式的请求驱动类型的轻量级Web框架，通过把模型-视图-控制器分离，将web层进行职责解耦，简化开发。

## 核心组件

### Spring MVC的主要组件？

（1）前端控制器 DispatcherServlet（不需要程序员开发）

作用：接收请求、响应结果，相当于转发器，有了DispatcherServlet 就减少了其它组件之间的耦合度。

（2）处理器映射器HandlerMapping（不需要程序员开发）

作用：根据请求的URL来查找Handler

（3）处理器适配器HandlerAdapter

注意：在编写Handler的时候要按照HandlerAdapter要求的规则去编写，这样适配器HandlerAdapter才可以正确的去执行Handler。

（4）处理器Handler（需要程序员开发）

（5）视图解析器 ViewResolver（不需要程序员开发）

作用：进行视图的解析，根据视图逻辑名解析成真正的视图（view）

（6）视图View（需要程序员开发jsp）

View是一个接口， 它的实现类支持不同的视图类型（jsp，freemarker，pdf等等）

### 什么是DispatcherServlet

Spring的MVC框架是围绕DispatcherServlet来设计的，它用来处理所有的HTTP请求和响应。

### 什么是Spring MVC框架的控制器？

控制器使用@Controller注解，并且提供了@RequestMapping注解，通过解析url中的路径，可以找到相应的控制器进行逻辑处理，响应数据等。

### Spring MVC的控制器是不是单例模式,如果是,有什么问题,怎么解决？

单例模式,所以在多线程访问的时候有线程安全问题,不要用同步,会影响性能的，解决方案是在控制器保持控制器无状态（不要写字段）。

## 工作原理

### 请描述Spring MVC的工作流程？描述一下 DispatcherServlet 的工作流程？

1. 用户发送请求至前端控制器DispatcherServlet；
2. DispatcherServlet收到请求后，调用HandlerMapping处理器映射器，请求获取Handler；
3. HandlerMapping处理器映射器根据请求url**找到具体的处理器**，生成处理器对象及处理器拦截器(如果有则生成)一并返回给DispatcherServlet；
4. DispatcherServlet 调用 HandlerAdapter处理器适配器；
5. HandlerAdapter 经过适配 **调用具体处理器**(Handler，也叫后端控制器)；
6. Handler执行完成返回ModelAndView；
7. HandlerAdapter将Handler执行结果ModelAndView返回给DispatcherServlet；
8. DispatcherServlet将ModelAndView传给ViewResolver视图解析器进行解析；
9. ViewResolver解析后返回具体View；
10. DispatcherServlet对View进行渲染成为视图（即将模型数据填充至视图中）；
11. DispatcherServlet响应用户。

![Spring MVC的工作流程](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotes20200208211439106.png)

## 常用注解

### 注解原理是什么

注解本质是一个继承了Annotation的特殊接口，其具体实现类是Java运行时生成的动态代理类。我们通过反射获取注解时，返回的是Java运行时生成的动态代理对象。通过代理对象调用自定义注解的方法。

### Spring MVC常用的注解有哪些？

@RequestMapping：用于处理请求 url 映射的注解，可用于类或方法上。用于类上，则表示类中的所有响应请求的方法都是以该地址作为父路径。

@RequestBody：注解实现接收http请求的json数据，将json转换为java对象。

@ResponseBody：注解实现将controller方法返回对象转化为json对象响应给客户。

### Sping MVC中的控制器的注解一般用哪个,有没有别的注解可以替代？

答：一般用@Controller注解，也可以使用@RestController，@RestController注解相当于@ResponseBody ＋ @Controller，@ResponseBody 用于返回json数据＋ @Controller 用于表明控制器

### @Controller注解的作用

@Controller 用于标记在一个类上，使用它标记的类就是一个Spring MVC Controller 对象。通过请求的url和@RequestMapping 注解对应到实际处理请求的处理器。需要开启mvc注解扫描找到@Controller的控制器。

### @RequestMapping注解的作用

RequestMapping是一个用来处理请求地址映射的注解，可用于类或方法上。

用于类上，表示类中的所有响应请求的方法都是以该地址作为父路径。

RequestMapping注解有六个属性：

**path/value， method**

* value： 指定请求的实际地址，指定的地址可以是URI Template 模式（后面将会说明）；

* method： 指定请求的method类型， GET、POST、PUT、DELETE等；

**consumes，produces**

* consumes： 指定处理请求的提交内容类型（Content-Type），例如application/json, text/html;

* produces: 指定返回的内容类型，仅当request请求头中的(Accept)类型中包含该指定类型才返回；

**params，headers**

* params： 指定request中必须包含某些参数值时，才让该方法处理。

* headers： 指定request中必须包含某些指定的header值，才能让该方法处理请求。

### @PathVariable和@RequestParam的区别

@PathVariable作用： 用于绑定url中的占位符。例如：请求url中 /delete/{id}，这个{id}就是url占位符。

```java
/**
* 根据path /anno/testPathVariable/10 找到 testPathVariable 方法
* 然后，其中的10就是sid占位符所占的位置的数据，然后通过@PathVaribale注解将其赋值给 id
*/
@RequestMapping(path = "testPathVariable/{sid}")
public String testPathVariable(@PathVariable(value = "sid") String id) {
    System.out.println("执行了testPathVariable...");
    System.out.println(id);
    return "success";
}
```

@RequestParam作用： 把请求中指定名称的参数给控制器中的形参赋值。

```java
// Controller中的代码：
@RequestMapping(path = "/testRequestParam")
public String testRequestParam(@RequestParam(value = "name") String username) {
    // 把请求中指定的name赋值给控制器中的形参username
    System.out.println("执行了testRequestParam...");
    System.out.println(username);
    return "success";
}
```

## 其他

### Spring MVC怎么样设定转发和重定向的？

（1）转发：使用forward
forward是指内部转发，相当于服务器内部方法跳转调用。请求一次服务器。转发地址不变。

（2）重定向：使用redirect
重定向是指当浏览器请求一个URL时，服务器返回一个重定向指令，告诉浏览器地址已经变了，麻烦使用新的URL再重新发送新请求。请求两次服务器。重定向地址改变。

### Spring MVC怎么和AJAX相互调用的？

通过Jackson或者是fastjson就可以把Java里面的对象直接转化成Js可以识别的Json对象，来进行传输。

### 如何解决POST请求中文乱码问题，GET的又如何处理呢？

（1）解决post请求乱码问题：

在web.xml中配置一个CharacterEncodingFilter过滤器，设置成utf-8；

```xml
<filter>
    <filter-name>CharacterEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>

    <init-param>
        <param-name>encoding</param-name>
        <param-value>utf-8</param-value>
    </init-param>
</filter>

<filter-mapping>
    <filter-name>CharacterEncodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

（2）get请求中文参数出现乱码解决方法有两个：

修改tomcat配置文件添加编码与工程编码一致，如下：

```xml
<ConnectorURIEncoding="utf-8" connectionTimeout="20000" port="8080" protocol="HTTP/1.1" redirectPort="8443"/>
```

### Spring MVC的异常处理

可以将异常抛给Spring框架，由Spring框架来处理；我们只需要配置简单的异常处理器，在异常处理器中添视图页面即可。

### 如果在拦截请求中，我想拦截get方式提交的方法,怎么配置

答：可以在@RequestMapping注解里面加上method=RequestMethod.GET。

### 怎样在方法里面得到Request,或者Session？

答：直接在方法的形参中声明request,Spring MVC就自动把request对象传入。

### 如果想在拦截的方法里面得到从前台传入的参数,怎么得到？

直接在形参里面声明这个参数就可以,但必须名字和传过来的参数一样。
或者使用 RequestParam注解将前台的参数名A传给形参中的参数B

### 如果前台有很多个参数传入,并且这些参数都是一个对象的,那么怎么样快速得到这个对象？

直接在方法中声明这个对象,Spring MVC就自动会把属性赋值到这个对象里面。

### Spring MVC中函数的返回值是什么？

返回值可以有很多类型,有String, ModelAndView（视图和数据合并在一起）。

### Spring MVC用什么对象从后台向前台传递数据的？

通过ModelMap对象,可以在这个对象里面调用put方法,把对象加到里面,前台就可以通过el表达式拿到。

### 怎么样把ModelMap里面的数据放入Session里面？

可以在类上面加上@SessionAttributes注解,里面包含的字符串就是要放入session里面的key。

### Spring MVC里面拦截器是怎么写的

有两种写法,一种是实现HandlerInterceptor接口，重写prehandle等方法，然后配置拦截器
