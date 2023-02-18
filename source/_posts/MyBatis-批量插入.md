---
title: MyBatis 批量插入
copyright: true
mathjax: false
categories:
  - 技术总结
  - MyBatis
toc: false
date: 2023-02-16 17:16:34
tags:
urlname: mybatis-batch-insert
---

本文对 MyBatis 批量插入做简单小记。<!--more-->

## 解决方案

首先对于批量数据的插入有两种解决方案

1、for 循环调用 Dao 中的单条插入方法

2、传一个 List\<Object> 参数，使用 MyBatis 的 foreach 标签进行批量插入

```xml
<insert id="addUser" parameterType="java.util.List" >
  insert into user(name,age) values
    <foreach collection="list" item="item" index="index" separator=",">
        (#{item.name},#{item.age})
    </foreach>
</insert>
```

性能上对比，批量插入性能好，更加省时间。

原因如下：

- 循环插入：需要每次都获取 session, 获取连接，然后将 SQL 语句发给 MySQL 去执行（JDBC 一般情况下是通过 TCP/IP 进行连接和数据库进行通信的）。可以看这里 [mysql 四种通信协议](https://my.oschina.net/zjllovecode/blog/1617754)
- 批量插入： 批量插入通过 foreach 标签，将多条数据拼接在 SQL 语句后，一次执行只获取一次 session, 提交一条 SQL 语句。减少了程序和数据库交互的准备时间。

## 注意点

批量插入有需要注意的地方：

### 返回值

**对于普通的单条插入，数据库的返回值就是 （0/1） 。**

对于返回值代表的意思可以认为是“语句执行返回的数据库受影响的行数。”或者是“此次执行是否成功（0 - 失败，1 - 成功）。”

对应的也就是在 Dao 层中，对于插入方法的返回值类型的设定有（int/boolean）两种。 

**对于批量插入的返回值，返回的还是（0/1）, 而不是统计插入成功几条，即使你的 Dao 层方法的返回值类型为 int.**

这里的（0/1） 也就代表着，这次批量插入是否成功（0 - 失败，1 - 成功）。

当然你 Dao 层的返回值还是可以是（int/boolean）

### 批量插入中间有一个失败会怎么样

猜想有下面三种情况

a、继续插入后面的，把失败的跳过

b、停止插入，但前面的插入成功保持。

c、全部回滚

这里就直接放结果了。

**批量语句，只要有一个失败，就会全部失败。数据库会回滚全部数据。（原子性）**

关于测试过程可以看这篇博客：[mysql 批量插入语句执行失败的话，是部分失败还是全部失败](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwww.cnblogs.com%2Fgrey-wolf%2Fp%2F7117036.html) 

其实也很好理解。

首先我们知道了 MyBatis \<foreach\> 批量插入，是在程序内拼接 SQL 语句（拼接成多条同时插入的 SQL 语句），拼接后发给数据库。

就相当于咱们自己在 MySQL 的命令行中，执行一条多插入的语句。默认情况下 MySQL 单条语句是一个事务，这在一个事务范围内，当中间的 SQL 语句有问题，或者有一个插入失败，就会触发事务回滚。同时你也能看到错误提示。（命令行执行单条 SQL 的情况）

所以有一个插入不成功肯定全部回滚。

### 批量插入数据量的限制

我这里就直接放结论，又兴趣的可以看这篇博客有探究过程 ： [Mybatis 批量插入引发的血案](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fblog.csdn.net%2Fsyy_c_j%2Farticle%2Fdetails%2F52151402)

**1、MyBatis 本身对插入的数据量没有限制**

**2、MySQL 对语句的长度有限制，默认是 4M**

其他数据库的情况这里不介绍，可以自行百度。通过上面 “MySQL 对语句的长度有限制，默认是 4M” 我们可以知道，批量插入数据是有限制的。不能一下把几万条数据（就是太大数据量意思）一次性插入。

**所以一般情况下我们推荐即使使用批量插入，也要分批次。**

每次批次设置多少？需要根据你的插入一条数据的参数量来做度量。因为受限条件是 SQL 语句的长度。

而且分批插入更加合理，对于插入失败，回滚范围会缩小很多。

### 对空集合参数进行校验

MyBatis 并没有做集合容量的验证，如果集合参数为空或者 size 为 0 则生成的 SQL 可能只有”insert into user (name,age) values” 这样一段或者没有，所以说，写批量 SQL 的时候注意在调用批量方法的地方加入对容量的验证。

### 另外一种 foreach 插入（不推荐）

```xml
<insert id="addBatchUser" parameterType="java.util.List" >
    <foreach collection="list" item="item" index="index" separator=";">
        insert into user(name,age) values(#{item.name},#{item.age})
    </foreach>
</insert> 
```

这种写法也能实现批量插入。但是有很多问题。

a、首先这种方式的批量插入也是 SQL 拼接。但是明显**字符长度增加。这就导致每批次可插入的数量减少**

b、这种方式执行返回值还是（0、1）是已经尝试插入的最后一条数据是否成功。由于这种 foreach 拼接成的 SQL 语句，是以分号 “；” 分隔的多条 insert 语句。这就导致前面的数据项都插入成功了。（**默认数据库的事务处理是单条提交的**，出错前的执行都是一个个单条语句，所以并并没有回滚数据。）

所以如果你想中间插入失败回滚的话，需要使用 Spring 事务，但是还需要注意 spring 事务是抛出运行时异常时才会回滚。这种批量插入中间有没插入成功的是不会抛出异常的。所以你需要根据返回值判断手动编码抛出异常。

而最上面的那种写法就不用是用事务，因为他是一条 SQL 语句。

## \<foreach\> 标签

foreach 的主要用在构建 in 条件中，它可以在 SQL 语句中进行迭代一个集合。

foreach 元素的属性主要有 item，index，collection，open，separator，close。
item 表示集合中每一个元素进行迭代时的别名
index 指定一个名字，用于表示在迭代过程中，每次迭代到的位置
open 表示该语句以什么开始
separator 表示在每次进行迭代之间以什么符号作为分隔符
close 表示以什么结束

在使用 foreach 的时候最关键的也是最容易出错的就是 collection 属性，该属性是必须指定的，但是在不同情况 下，该属性的值是不一样的，主要有一下 3 种情况：

1、如果传入的是单参数且参数类型是一个 List 的时候，collection 属性值为 list

2、如果传入的是单参数且参数类型是一个 array 数组的时候，collection 的属性值为 array

3、如果传入的参数是多个的时候，我们就需要把它们封装成一个 Map 了，当然单参数也可以封装成 map

## 参考

https://my.oschina.net/zjllovecode/blog/1818716
