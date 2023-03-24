---
title: "Java八股文 - Spring"
date: 2023-01-23 16:14:06
categories: ["Java八股文"]
tags: ["Spring"]
url: java-spring

################################目录################################
toc: true
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

> 整理的Spring框架相关知识点和面试题，部分内容摘自网络，如有侵权请联系我～	<!--more-->

## Spring概述

### 什么是spring?

Spring 是**一个轻量级Java开发框架**，目的是为了解决开发中的**业务逻辑层和其他各层的耦合问题**，简化 Java 开发。

### Spring框架的核心是什么

**Spring框架的核心**：IoC 容器和 AOP 模块。
通过 IoC 容器管理 Java Bean 对象及其生命周期以及他们之间的耦合关系；
通过 AOP 将一些公共行为和逻辑，抽取并封装为一个可重用的模块，减少系统中的重复代码，降低模块间的耦合度

### Spring的优缺点是什么？

优点

- 方便解耦，简化开发（**IoC**）

	Spring 就是一个大工厂，可以将所有对象的创建和依赖关系的维护，交给 Spring 管理。

- AOP 编程的支持（**AOP**）

	Spring 提供面向切面编程，可以方便的实现对程序进行权限拦截、运行监控等功能。

- 声明式事务的支持（**事务**）

	只需要通过配置就可以完成对事务的管理，而无需手动编程。

- 方便集成各种优秀框架（**集成**）

	Spring 不排斥各种优秀的开源框架，其内部提供了对各种优秀框架的直接支持（如：Struts、Hibernate、MyBatis 等）。


缺点

- Spring**依赖反射，反射影响性能**

### Spring 框架中都用到了哪些设计模式？

1. 工厂模式：通过 BeanFactory 和 ApplicationContext 来管理 Bean 对象。
2. 单例模式：Bean 默认为单例模式。
3. 代理模式：Spring 的 AOP 功能用到了基于接口的 JDK 的动态代理和基于子类的 CGLIB 动态代理；

### Spring框架中有哪些不同类型的事件

**Spring的ApplicationContext 提供了支持事件和代码中监听器的功能。**  

如果一个 bean 实现了`ApplicationListener`接口，当一个`ApplicationEvent` 被发布以后，bean 会自动被通知。

```javascript
public class AllApplicationEventListener implements ApplicationListener < ApplicationEvent >{
    @Override
    public void onApplicationEvent(ApplicationEvent applicationEvent){
        //process event
    }
}
```

**Spring 提供了以下5种标准的事件：**

- 上下文更新事件（ContextRefreshedEvent）：该事件会在 ApplicationContext 被初始化或者更新时发布。也可以在调用`ConfigurableApplicationContext` 接口中的`refresh()`方法时被触发。
- 上下文开始事件（ContextStartedEvent）：当容器调用`ConfigurableApplicationContext`的`Start()`方法开始/重新开始容器时触发该事件。
- 上下文停止事件（ContextStoppedEvent）：当容器调用`ConfigurableApplicationContext`的`Stop()`方法停止容器时触发该事件。
- 上下文关闭事件（ContextClosedEvent）：当`ApplicationContext`被关闭时触发该事件。容器被关闭时，其管理的所有单例 Bean 都被销毁。
- 请求处理事件（RequestHandledEvent）：在 Web 应用中，当一个 http 请求（request）结束触发该事件。

### Spring 应用程序有哪些不同组件？

Spring 应用一般有以下组件：

- Bean 类 - 它包含属性，setter 和 getter 方法，函数等。
- Bean 配置文件 - 包含类的信息以及如何配置它们。
- 接口 - 处理 Bean 
- 用户程序 - 它使用接口。

## Spring控制反转(IOC)

### 什么是Spring IOC 容器？

Spring IOC（控制反转） 把组件对象控制权交给容器，容器负责创建对象，配置对象（通过依赖注入（DI），并且管理这些对象的整个生命周期。IoC 让相互协作的组件保持松耦合。

### Spring IoC 的实现机制

Spring 中的 IoC 的实现原理就是**工厂模式加反射**。

```java
interface Fruit {
    void eat();
}

class Apple implements Fruit {
    public void eat(){
        System.out.println("Apple");
    }
}

class Orange implements Fruit {
    public void eat(){
        System.out.println("Orange");
    }
}

class Factory {
    public static Fruit getInstance(String className) {	// 使用反射，用字符串获取到实例
        Fruit f=null;
        try {
            f = (Fruit) Class.forName(className).newInstance();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return f;
    }
}

class Client {
    public static void main(String[] a) {
        Fruit f = Factory.getInstance("io.github.dunwu.spring.Apple");
        if(f != null){
            f.eat();
        }
    }
}
```

### BeanFactory 和 ApplicationContext有什么区别？

BeanFactory 和 ApplicationContext 是 Spring 的两大核心接口，都可以当做 Spring 的容器。
ApplicationContext 是 BeanFactory 的子接口。

1. 依赖关系

BeanFactory：是 Spring 里面最底层的接口，包含了各种 Bean 的定义，读取 bean 配置文档，管理 bean 的加载、实例化，控制 bean 的生命周期，维护 bean 之间的依赖关系。

ApplicationContext 接口作为 BeanFactory 的子接口，除了提供 BeanFactory 所具有的功能外，还提供了更完整的框架功能：

- 支持国际化。
- 支持访问文件资源
- 支持事件发布通知
- 同时加载多个配置文件。

配置流程：

1. 加载配置文件，解析成 BeanDefinition 放在 Map 里，map 中存放`<BeanName，Class对象>`的映射
2. 调用 getBean 的时候，从 BeanDefinition 所属的 Map 里，拿出 Class 对象使用反射进行实例化，同时，如果有依赖关系，将递归调用 getBean 方法 —— 完成依赖注入。
3. 加载方式

**BeanFactroy**采用的是**延迟加载**形式来注入 Bean 的，即只有在使用到某个 Bean 时(调用 getBean())，才对该 Bean 进行加载实例化。这样，我们就不能发现一些存在的 Spring 的配置问题。如果 Bean 的某一个属性没有注入或者是属性注入错误，BeanFacotry 加载后，直至第一次使用调用 getBean 方法才会抛出异常。

**ApplicationContext**，它是在容器启动时，**一次性创建**了所有的 Bean。这样，在容器启动时，我们就可以**发现Spring中存在的配置错误**，这样有利于检查所依赖属性是否注入。 ApplicationContext 启动后预载入所有的单实例 Bean，通过预载入单实例 bean ,确保当你需要的时候，你就不用等待，因为它们已经创建好了。

由于在容器启动时一次性创建所有的 Bean，因此 ApplicationContext 更加占用内存空间，当配置的 Bean 比较多时，程序启动较慢。

#### ⭐BeanFactory和FactoryBean区别？

BeanFactory：是 spring IoC 容器的底层接口，可以用来管理 bean 及其生命周期；

FactoryBean：如果某个 bean 实现了 FactoryBean 这个接口，通过 getBean 方法来获取 bean 的时候，并不是返回自己的实例，而是返回其 getObject() 方法的返回值

### ApplicationContext通常的实现是什么？

**FileSystemXmlApplicationContext** ：此容器从一个 XML 文件中加载 beans 的定义，XML Bean 配置文件的全路径名必须提供给它的构造函数。

**ClassPathXmlApplicationContext**：此容器也从一个 XML 文件中加载 beans 的定义，这里，你需要正确设置 classpath 因为这个容器将在 classpath 里找 bean 配置。

**AnnotationConfigApplicationContext**：它是用于读取注解创建容器的

### 什么是Spring的依赖注入？

具体含义是:当某个调用者需要另一个被调用者的协助时，在 传统的程序设计过程中，通常由调用者来创建被调用者的实例。但在 Spring 里，创建被调用者的工作不再由调用者来完成，因此称为控制反转；创建被调用者 实例的工作通常由 Spring 容器来完成，然后注入调用者，因此也称为依赖注入。

### 依赖注入有什么优势

依赖注入之所以更流行是因为它是一种更可取的方式：让容器全权负责依赖查询，受管组件只需要暴露 JavaBean 的 setter 方法或者带参数的构造器或者接口，使容器可以在初始化时组装对象的依赖关系。其与依赖查找方式相比，主要优势为：

- 查找定位操作与应用代码完全无关。
- 不依赖于容器的 API，可以很容易地在任何容器以外使用应用对象。
- 不需要特殊的接口，绝大多数对象可以做到完全不必依赖容器。

### 有哪些不同类型的依赖注入实现方式？

**构造器依赖注入**：构造器依赖注入通过容器触发一个类的构造器来实现的，该类有一系列参数，每个参数代表一个对其他类的依赖。

**Setter方法注入**：Setter 方法注入是容器通过调用无参构造器或无参 static 工厂 方法实例化 bean 之后，调用该 bean 的 setter 方法，即实现了基于 setter 的依赖注入。

### 构造器依赖注入和 setter方法注入的区别

（部分注入）在 setter 注入,可以将依赖项部分注入,构造方法注入不能部分注入。

（属性覆盖）如果我们为同一属性提供 setter 和构造方法注入，setter 注入将覆盖构造方法注入。但是构造方法注入不能覆盖 setter 注入值。显然，构造方法注入被称为创建实例的第一选项。

（循环依赖）在构造函数注入,如果 A 和 B 对象相互依赖：A 依赖于 B,B 也依赖于 A,此时在创建对象的 A 或者 B 时，Spring 抛出 ObjectCurrentlyInCreationException。所以 Spring 可以通过 setter 注入,从而解决循环依赖的问题。

**最好的解决方案是用构造器参数实现强制依赖，setter方法实现可选依赖**

### 讲一讲Spring中的循环依赖

https://www.cnblogs.com/daimzh/p/13256413.html

https://blog.csdn.net/weixin_49592546/article/details/108050566

#### 什么是循环依赖？

```java
@Component
public class A {
    // A中注入了B
	@Autowired
	private B b;
}

@Component
public class B {
    // B中也注入了A
	@Autowired
	private A a;
}
```

#### Spring是如何解决的循环依赖？

> 以下：
>
> 作者：阿里云云栖号
>
> 链接：https://zhuanlan.zhihu.com/p/368769721
>
> 来源：知乎
>
> 著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

首先，Spring 解决循环依赖有两个前提条件：

1. 不全是构造器方式的循环依赖
2. 必须是单例

基于上面的问题，我们知道 Bean 的生命周期，本质上解决循环依赖的问题就是三级缓存，通过三级缓存提前拿到未初始化完全的对象。

```java
/** Cache of singleton objects: bean name to bean instance. */
private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);
/** Cache of early singleton objects: bean name to bean instance. */
private final Map<String, Object> earlySingletonObjects = new HashMap<>(16);
/** Cache of singleton factories: bean name to ObjectFactory. */
private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>(16);
```

第一级缓存：用来保存实例化、初始化都完成的对象

第二级缓存：用来保存实例化完成，但是未初始化完成的对象

第三级缓存：用来保存一个对象工厂，提供一个匿名内部类，用于创建二级缓存中的对象

![Spring三级缓存](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotesv2-dbda6a103e41cde4a54548386d6c1cc2_720w.jpg)

假设一个简单的循环依赖场景，A、B 互相依赖。

![简单的循环依赖场景](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotesv2-f20f1e81c8011fd99f723127cacf7430_720w.jpg)

A 对象的创建过程：

1. 创建对象 A，实例化的时候把 A 对象工厂放入三级缓存；

![实例化对象A](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotesv2-b295374aff4235ee9c0538bf03856217_720w.jpg)



2. A 注入属性时，发现依赖 B，转而去实例化 B

3. 同样创建对象 B，注入属性时发现依赖 A，依次从一级到三级缓存查询 A，从三级缓存通过对象工厂拿到 A，把 A 放入二级缓存，同时删除三级缓存中的 A，此时，B 已经实例化并且初始化完成，把 B 放入一级缓存。

![创建对象B](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotesv2-51cb735fc6c02c2225d2f38b9b5b6da9_720w.jpg)

4. 接着继续创建 A，顺利从一级缓存拿到实例化且初始化完成的 B 对象，A 对象创建也完成，删除二级缓存中的 A，同时把 A 放入一级缓存

5. 最后，一级缓存中保存着实例化、初始化都完成的 A、B 对象

![最终结果](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotesv2-a1eaa5d9f8bec23e373dc1b9f86ddb25_720w.jpg)

因此，由于把实例化和初始化的流程分开了，所以如果都是用构造器的话，就没法分离这个操作，所以都是构造器的话就无法解决循环依赖的问题了。

### 8. 为什么要三级缓存？二级不行吗？

不可以，主要是为了生成代理对象。

因为三级缓存中放的是生成具体对象的对象工厂，他可以生成代理对象，也可以是普通的实例对象。

假设只有二级缓存的情况，往二级缓存中放的显示一个普通的 Bean 对象，`BeanPostProcessor`去生成代理对象之后，覆盖掉二级缓存中的普通 Bean 对象，那么多线程环境下可能取到的对象就不一致了。

![A代理对象覆盖A普通Bean](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotesv2-e4e9050787017c7272dc313f88af9382_720w.jpg)



## Spring Beans

### 如何给Spring 容器提供配置元数据？Spring有几种配置方式

这里有三种重要的方法给 Spring 容器提供配置元数据。

- XML 配置文件。
- 基于注解的配置。
- 基于 Java 的配置。

### Spring配置文件包含了哪些信息

Spring 配置文件是个 XML 文件，通过读取指定类的全限定类名。反射创建对象，丢入 ioc 容器中，该对象可以通过 id 来获取

### Spring基于xml注入bean的几种方式

1. Set 方法注入：`<property name="xxx" value="yyy"/>`
2. 构造器注入：`<construtor-arg type/index/name="xxx" value="yyy"/>`
3. 静态工厂注入；factory-bean factory-method
4. 实例工厂； class factory-method

### 解释Spring支持的几种bean的作用域

Spring 框架支持以下五种 bean 的作用域：

- **singleton :** bean 在每个 Spring ioc 容器中只有一个实例。
- **prototype**：一个 bean 的定义可以有多个实例。
- **request**：每次 http 请求都会创建一个 bean，该作用域仅在基于 web 的 Spring ApplicationContext 情形下有效。
- **session**：在一个 HTTP Session 中，一个 bean 定义对应一个实例。该作用域仅在基于 web 的 Spring ApplicationContext 情形下有效。
- **global-session**：在一个全局的 HTTP Session 中，一个 bean 定义对应一个实例。该作用域仅在基于 web 的 Spring ApplicationContext 情形下有效。

**注意：** 缺省的 Spring bean 的作用域是 singleton。使用 prototype 作用域需要慎重的思考，因为频繁创建和销毁 bean 会带来很大的性能开销。

### Spring框架中的单例bean是线程安全的吗？

不是，Spring 框架中的单例 bean 不是线程安全的。

spring 中的 bean 默认是单例模式，spring 框架并没有对单例 bean 进行多线程的封装处理。

然而，实际上大部分时候 spring bean 无状态的（比如 dao 类），所以某种程度上来说 bean 也是安全的，但如果 bean 有状态的话（比如 view model 对象），那就要开发者自己去保证线程安全了，最简单的就是改变 bean 的作用域，把“singleton”变更为“prototype”，这样请求 bean 相当于 new Bean()了，所以就可以保证线程安全了。

> 有状态就是有数据存储功能
> 无状态就是不会保存数据

### Spring如何处理线程并发问题？

**Spring使用ThreadLocal对一些Bean的线程安全问题进行处理**。

ThreadLocal 和线程同步机制都是为了解决多线程中相同变量的访问冲突问题。同步机制采用了“时间换空间”的方式，仅提供一份变量，不同的线程在访问前需要获取锁，没获得锁的线程则需要排队。而 ThreadLocal 采用了“空间换时间”的方式。

ThreadLocal 会为每一个线程提供一个独立的变量副本，从而隔离了多个线程对数据的访问冲突。因为每一个线程都拥有自己的变量副本，从而也就没有必要对该变量进行同步了。ThreadLocal 提供了线程安全的共享对象，在编写多线程代码时，可以把不安全的变量封装进 ThreadLocal。

### 解释Spring框架中bean的生命周期

参考：https://www.cnblogs.com/javazhiyin/p/10905294.html

![Spring中Bean的生命周期](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotes201911012343410.png)

**1、实例化Bean：**

对于 BeanFactory 容器，当客户向容器请求一个尚未初始化的 bean 时，或初始化 bean 的时候需要注入另一个尚未初始化的依赖时，容器就会调用 createBean 进行实例化。
对于 ApplicationContext 容器，当容器启动结束后，通过获取 BeanDefinition 对象中的信息，实例化所有的 bean。

**2、依赖注入：**

实例化后的对象被封装在 BeanWrapper 对象中，紧接着，Spring 根据 BeanDefinition 中的信息 以及 通过 BeanWrapper 提供的设置属性的接口完成依赖注入。

**3、处理Aware接口（配置Bean对象的id值、工厂/上下文）：**

接着，Spring 会检测该对象是否实现了 xxxAware 接口，并将相关的 xxxAware 实例注入给 Bean：

- 如果这个 Bean 已经实现了 BeanNameAware 接口，会调用它实现的 setBeanName(String beanId)方法，此处传递的就是 Spring 配置文件中 Bean 的 id 值；
- 如果这个 Bean 已经实现了 BeanFactoryAware 接口，会调用它实现的 setBeanFactory()方法，传递的是 Spring 工厂自身；
- 如果这个 Bean 已经实现了 ApplicationContextAware 接口，会调 setApplicationContext(ApplicationContext)方法，传入 Spring 上下文；

**4、postProcessBeforeInitialization（初始化前置处理）：**

如果想对 Bean 进行一些自定义的处理，那么可以让 Bean 实现了 BeanPostProcessor 接口，那将会调用 postProcessBeforeInitialization(Object obj, String s)方法。由于这个方法是在 Bean 初始化结束时调用的，所以可以被应用于内存或缓存技术；

**5、自定义初始化init-method：**

如果 Bean 在 Spring 配置文件中配置了 init-method 属性，则会自动调用其配置的**自定义初始化**方法。

**6、postProcessAfterInitialization（初始化后置处理）**

如果这个 Bean 实现了 BeanPostProcessor 接口，将会调用 postProcessAfterInitialization(Object obj, String s)方法，**AOP在这个时候进行代理对象的创建**。

**NOW** 以上几个步骤完成后，Bean 就已经被正确创建了，之后就可以使用这个 Bean 了。

**7、清理阶段destroy：**

当 Bean 不再需要时，会进入清理阶段，如果 Bean 实现了 DisposableBean 这个接口，会调用其实现的 destroy()方法；

**8、自定义销毁destroy-method：**

最后，如果这个 Bean 的 Spring 配置中配置了 destroy-method 属性，会自动调用其配置的**自定义销毁**方法。

### 在 Spring中如何注入一个Java集合？

用`<list>`注入一列值，用`<map>`注入一组映射数据。

### 什么是bean装配？

通过 bean 的依赖关系，使用依赖注入将 spring 中的 bean 装配在一起。
spring 可以通过 bean 的依赖关系自动完成 bean 之间的配置。

### Spring 自动装配 bean 有哪些方式？

在 spring 中，对象无需自己查找或创建与其关联的其他对象，由容器负责把需要相互协作的对象引用赋予各个对象。

在 Spring 框架 xml 配置中共有 5 种自动装配：

- no：默认的方式是不进行自动装配的，通过手工设置 ref 属性来进行装配 bean。
- byName：通过 bean 的名称进行自动装配，如果一个 bean 的 property 与另一 bean 的 name 相同，就进行自动装配。
- byType：通过参数的数据类型进行自动装配。
- constructor：利用构造函数进行装配，并且构造函数的参数通过 byType 进行装配。
- autodetect：自动探测，如果有构造方法，通过 construct 的方式自动装配，否则使用 byType 的方式自动装配。

### 使用@Autowired注解自动装配的过程是怎样的？

使用@Autowired 注解来自动装配指定的 bean。在使用@Autowired 注解之前需要在 Spring 配置文件进行配置注解扫描 `<context:annotation-config />`。

在启动 spring IoC 时，容器自动装载了一个后置处理器，当容器扫描到@Autowied、@Resource 或@Inject 时，就会在 IoC 容器自动查找需要的 bean，并装配给该对象的属性。在使用@Autowired 时，首先在容器中**查询对应类型**的 bean：

- 如果对应类型查询结果刚好为一个，就将该 bean 装配给@Autowired 指定的数据；
- 如果对应类型查询的结果不止一个，那么@Autowired 会根据名称来查找；
- 如果上述查找的结果为空或者不止一个，那么会抛出异常。

## Spring注解

### 什么是基于Java的Spring注解配置? 给一些注解的例子

基于 Java 的配置，允许你在少量的 Java 注解的帮助下，进行你的大部分 Spring 配置而非通过 XML 文件。

以@Configuration 注解为例，它用来标记类可以当做一个 bean 的定义，被 Spring IOC 容器使用。

另一个例子是@Bean 注解，它表示此方法将要返回一个对象，作为一个 bean 注册进 Spring 应用上下文。

```java
@Configuration
public class StudentConfig {
    @Bean
    public StudentBean myStudent() {
        return new StudentBean();
    }
}
```

### 怎样开启注解装配？

注解装配在默认情况下是不开启的，为了使用注解装配，必须在 Spring 配置文件中配置 `<context:annotation-config/>`。

### @Component, @Controller, @Repository, @Service 有何区别？

@Component：这将 Java 类标记为 bean。spring 的组件扫描机制可以将其扫描并将其注册到 IoC 容器中。

@Controller：这将一个类标记为 Spring Web MVC 控制器。标有它的 Bean 会自动导入到 IoC 容器中。

@Service：此注解是组件注解的特化。它不会对 @Component 注解提供任何其他行为。您可以在服务层类中使用 @Service 而不是 @Component，因为它以更好的方式指定了意图。

@Repository：这个注解是具有类似用途和功能的 @Component 注解的特化。它为 DAO 提供了额外的好处。它将 DAO 导入 IoC 容器，并使未经检查的异常有资格转换为 Spring DataAccessException。

### @Required 注解有什么作用

`@Required`注解作用于 Bean`setter`方法上，用于检查一个 Bean 的属性的值**在配置期间是否被赋予或设置**，如果未被设置，容器将抛出 BeanInitializationException。示例：

```java
public class Employee {
    private String name;
    @Required
    public void setName(String name){
        this.name=name;
    }
    public string getName(){
        return name;
    }
}
```

### @Autowired和@Resource之间的区别

@Autowired 可用于：构造函数、成员变量、Setter 方法

@Autowired 和@Resource 之间的区别

- @Autowired 默认是**按照类型装配注入**的，默认情况下它要求依赖对象必须存在（可以设置它 required 属性为 false）。
- @Resource 默认是**按照名称来装配注入**的，只有当找不到与名称匹配的 bean 才会按照类型来装配注入。

### @Qualifier 注解有什么作用

当您创建多个相同类型的 bean 并希望仅使用属性装配其中一个 bean 时，您可以使用@Qualifier 注解和 @Autowired 通过指定应该装配哪个确切的 bean 来消除歧义。

### @RequestMapping 注解有什么用？

@RequestMapping 注解用于将特定 HTTP 请求方法映射到将处理相应请求的控制器中的特定类/方法。此注释可应用于两个级别：

- 类级别：映射请求的 URL
- 方法级别：映射 URL 以及 HTTP 请求方法

## Spring事务

### Spring支持的事务管理类型/事务实现方式有哪些？

Spring 支持两种类型的事务管理：

**编程式事务管理**：这意味你通过编程的方式管理事务，给你带来极大的灵活性，但是难维护。

**声明式事务管理**：这意味着你可以将业务代码和事务管理分离，你只需用注解和 XML 配置来管理事务。

### Spring事务的实现方式和实现原理

Spring 事务的本质其实就是数据库对事务的支持，没有数据库的事务支持，Spring 是无法提供事务功能的。

```xml
<!-- 2、配置事务的通知以及事务的属性-->
<tx:advice id="txAdvice" transaction-manager="transactionManager">
<!-- 配置事务的属性
isolation：用于指定事务的隔离级别。默认值是DEFAULT，表示使用数据库的默认隔
离级别。
propagation：用于指定事务的传播行为。默认值是REQUIRED(增删改)，表示一定会
有事务。查询方法可以选择SUPPORTS。
read-only：用于指定事务是否只读。只有查询方法才能设置为true。默认值是
false，表示读写。
timeout：用于指定事务的超时时间，默认值是-1，表示永不超时。如果指定了数值，
以秒为单位。
rollback-for：用于指定一个异常，当产生该异常时，事务回滚，产生其他异常时，
事务不回滚。没有默认值。表示任何异常都回滚。
no-rollback-for：用于指定一个异常，当产生该异常时，事务不回滚，产生其他异常
时事务回滚。没有默认值。表示任何异常都回滚。
-->
<tx:attributes>
    <!--非查询方法-->
    <tx:method name="*" propagation="REQUIRED" read-only="false"/>
    <!--查询方法-->
    <tx:method name="find*" propagation="SUPPORTS" read-only="true"/>
    </tx:attributes>
</tx:advice>
```

### 说一下Spring的事务传播行为

spring 事务的传播行为说的是，当多个事务同时存在的时候，spring 如何处理这些事务的行为。

> ① **PROPAGATION_REQUIRED**：如果当前没有事务，就创建一个新事务，如果当前存在事务，就加入该事务，该设置是最常用的设置。（默认值，适用于非查询方法，增删改）（事务存在就用不存在要创建）
> ② **PROPAGATION_SUPPORTS**：支持当前事务，如果当前存在事务，就加入该事务，如果当前不存在事务，就以非事务执行。（适用于查询方法）（事务存在就用不存在就不用）
> ③ PROPAGATION_MANDATORY：支持当前事务，如果当前存在事务，就加入该事务，如果当前不存在事务，就抛出异常。
> ④ **PROPAGATION_REQUIRES_NEW**：创建新事务，无论当前存不存在事务，都创建新事务。（永远都要新事务）
> ⑤ PROPAGATION_NOT_SUPPORTED：以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。
> ⑥ PROPAGATION_NEVER：以非事务方式执行，如果当前存在事务，则抛出异常。
> ⑦ PROPAGATION_NESTED：如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则按REQUIRED属性执行。

### 说一下 spring 的事务隔离？

spring 有五大隔离级别，默认值为 ISOLATION_DEFAULT（使用数据库的设置），其他四个隔离级别和数据库的隔离级别一致：

1. ISOLATION_DEFAULT：用底层数据库的设置隔离级别，数据库设置的是什么我就用什么；
2. ISOLATION_READ_UNCOMMITTED：读未提交，最低隔离级别、事务未提交前，就可被其他事务读取（会出现脏读、不可重复读、幻读）；
3. ISOLATION_READ_COMMITTED：读已提交，一个事务提交后才能被其他事务读取到（会造成不可重复读、幻读），SQL server 的默认级别；
4. ISOLATION_REPEATABLE_READ：可重复读，保证多次读取同一个数据时，其值都和事务开始时候的内容是一致，禁止读取到别的事务未提交的数据（会造成幻读），MySQL 的默认级别；
5. ISOLATION_SERIALIZABLE：序列化，代价最高最可靠的隔离级别，该隔离级别能防止脏读、不可重复读、幻读。

## Spring面向切面编程(AOP)

### 什么是AOP

OOP(Object-Oriented Programming)面向对象编程，允许开发者定义纵向的关系，但并适用于定义横向的关系，导致了大量代码的重复，而不利于各个模块的重用。

AOP(Aspect-Oriented Programming)，一般称为面向切面编程，作为面向对象的一种补充，将一些**公共行为和逻辑，抽取并封装为一个可重用的模块，减少系统中的重复代码，降低了模块间的耦合度，同时提高了系统的可维护性。可用于权限认证、日志、事务处理等**。

### Spring AOP and AspectJ AOP 有什么区别？AOP 有哪些实现方式？

AOP 实现的关键在于 代理模式，AOP 代理主要分为静态代理（AspectJ）和动态代理（Spring AOP）。

（1）AspectJ 采用静态代理模式，属于**编译时增强**，会在编译阶段织入切面生成相应的代理对象

（2）Spring AOP 采用动态代理模式，属于**运行时增强**，每次运行时织入切面在内存中临时生成相应的代理对象

### JDK动态代理和CGLIB动态代理的区别

JDK 动态代理主要是针对类实现了某个接口，AOP 则会使用 JDK 动态代理。基于反射的机制实现，生成一个实现该接口的一个代理类，然后通过重写方法的方式，实现对代码的增强。

而如果某个类没有实现接口，AOP 则会使用 CGLIB 代理。底层原理是**基于 asm 第三方框架**，通过修改字节码生成一个子类，然后重写父类的方法，实现对代码的增强。

> 作者：阿里云云栖号
>
> 链接：https://zhuanlan.zhihu.com/p/368769721
>
> 来源：知乎
>
> 著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

### 解释一下Spring AOP里面的几个名词

（1）切面（Aspect）：切面是通知和切点的结合。 在 Spring AOP 中，切面可以使用通用类（基于模式的风格） 或者在普通类中以 @AspectJ 注解来实现。

（2）连接点（Join point）：方法

（3）通知（Advice）：在要增强上的连接点上要进行怎样的增强

（4）切入点（Pointcut）：哪些连接点需要增强

（5）目标对象（Target Object）： 被代理（proxied） 对象。

（6）织入（Weaving）：织入是把切面应用到目标对象并创建新的代理对象的过程。

（7）Proxy（代理）：一个类被 AOP 织入增强后，就产生一个结果代理类。

> - 编译期：切面在目标类编译时被织入。AspectJ的织入编译器是以这种方式织入切面的。
> - 类加载期：切面在目标类加载到JVM时被织入。需要特殊的类加载器，它可以在目标类被引入应用之前增强该目标类的字节码。AspectJ5的加载时织入就支持以这种方式织入切面。
> - 运行期：切面在应用运行的某个时刻被织入。一般情况下，在织入切面时，AOP容器会为目标对象动态地创建一个代理对象。SpringAOP就是以这种方式织入切面。

### Spring通知有哪些类型？

在 AOP 术语中，切面的工作被称为通知，实际上是程序执行时要通过 SpringAOP 框架触发的代码段。

Spring 切面可以应用 5 种类型的通知：

1. 前置通知（Before）：在目标方法被调用之前调用通知功能；
2. 后置通知（After）：在目标方法完成之后调用通知，此时不会关心方法的输出是什么；
3. 返回通知（After-returning ）：在目标方法成功执行之后调用通知；
4. 异常通知（After-throwing）：在目标方法抛出异常后调用通知；
5. 环绕通知（Around）：通知包裹了被通知的方法，在被通知的方法调用之前和调用之后执行自定义的行为。

> 同一个aspect，不同advice的执行顺序：
>
> ①没有异常情况下的执行顺序：
>
> around before advice	方法调用之前自定义行为
> before advice
> target method 执行
> around after advice	方法调用后自定义行为
> after advice
> afterReturning
>
> ②有异常情况下的执行顺序：
>
> around before advice
> before advice
> target method 执行
> around after advice
> after advice
> afterThrowing:异常发生
> Java.lang.RuntimeException: 异常发生

### 什么是切面 Aspect？

aspect 由 pointcount 和 advice 组成，切面是通知和切点的结合。 
AOP 的工作重心在于如何将增强编织目标对象的连接点上, 这里包含两个工作:  

- 如何通过 pointcut 和 advice 定位到特定的 joinpoint 上
- 如何在 advice 中编写切面代码

可以简单地认为, 使用 @Aspect 注解的类就是切面

![在这里插入图片描述](https://yaxingfang-typora.oss-cn-hangzhou.aliyuncs.com/TyporaNotes2020021212264438.png)
