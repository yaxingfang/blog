---
title: "Java八股文 - MyBatis"
date: 2023-01-23 15:30:00
categories: [Java八股文"]
tags: ["MyBatis"]
url: java-mybatis

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

> 整理的MyBatis框架相关知识点和面试题，部分内容摘自网络，如有侵权请联系我～<!--more-->


### 什么是MyBatis？

1. MyBatis 是一个半 ORM（对象关系映射）框架，它内部封装了 JDBC，开发时只需要关注 SQL 语句本身，不需要花费精力去处理**加载驱动、创建连接、创建statement**等繁杂的过程。
2. 通过 xml 文件或注解的方式将要执行的各种 statement 配置起来，并通过 java 对象和 statement 中 SQL 的动态参数进行映射生成最终执行的 SQL 语句，最后由 mybatis 框架执行 SQL 并将结果映射为 java 对象并返回。（从执行 SQL 到返回 result 的过程）。

### ORM是什么

ORM（Object Relational Mapping），对象关系映射，就是把数据库表和实体类及实体类的属性对应起来

让我们可以操作实体类就实现操作数据库表。

### 为什么说MyBatis是半自动ORM映射工具？它与全自动的区别在哪里？

MyBatis 在查询关联对象或关联集合对象时，需要手动编写 SQL 来完成，所以，称之为半自动 ORM 映射工具。

### JDBC编程有哪些不足之处，MyBatis是如何解决的？

1. 数据库连接创建、释放频繁，造成系统资源浪费从而影响系统性能
2. SQL 语句写在代码中造成代码**不易维护**
3. 向 SQL 语句**传参**麻烦
4. 对**结果集解析**麻烦

### MyBatis编程步骤是什么样的？

1. 读取配置文件，创建 SqlSessionFactory 工厂
2. 通过 SqlSessionFactory 工厂创建 SqlSession 对象
3. 使用 SqlSession 创建 Dao 接口的代理对象
4. 使用代理对象执行方法
5. 调用 session.commit()提交事务
6. 调用 session.close()关闭会话

### \#{}和\${}的区别是什么？

mybatis 在处理 \#{} 时，会将 SQL 中的 \#{} 替换为 ? 号，调用 PreparedStatement 的 set 方法来赋值。
这是**预编译处理**，可以提高 SQL 执行效率，更重要的是**可以防止SQL注入**；

mybatis 在处理 \${} 时，就是把 \${} 使用**字符串替换**。使用该方式如果传入 id 的话，会把 id=1 转换为 id="1"字符串形式，而数据库中 id 实际为数值型这样就不匹配了，查询失败。（\${}是字符串替换，通常在 EL 表达式中使用，用来展示后端传递到前端的值）。

### 当实体类中的属性名和表中的字段名不一样 ，怎么办 ？

1. 通过在查询的 SQL 语句中定义数据库字段名的别名，让数据库字段名的别名和实体类的属性名一致。
2. 通过 `<resultMap>`来映射字段名和实体类属性名的一一对应的关系。

### 模糊查询like语句该怎么写?

1. 在 Java 代码中添加 SQL 通配符。

	```xml
	string wildcardname = “%smi%”;
	list<name> names = mapper.selectlike(wildcardname);
	
	<select id=”selectlike”>
		select * from foo where bar like #{value}
	</select>
	```

2. 在 SQL 语句中拼接通配符%，会引起 SQL 注入

	```xml
	string wildcardname = “smi”;
	list<name> names = mapper.selectlike(wildcardname);
	
	<select id=”selectlike”>
		select * from foo where bar like "%"#{value}"%"
	</select>
	```

### MyBatis都有哪些Executor执行器？它们之间的区别是什么？

MyBatis 有三种基本的 Executor 执行器，SimpleExecutor、ReuseExecutor、BatchExecutor。

- **SimpleExecutor** ：每次开启一个 Statement 对象，用完立刻关闭 Statement 对象。
- **ReuseExecutor** ：重复使用创建的 Statement 对象。
- **BatchExecutor** ：缓存了多个 Statement 对象，一起执行批量更新。

### 通常一个xml映射文件，都会写一个Dao接口与之对应，请问，这个Dao接口的工作原理是什么？Dao接口里的方法，参数不同时，方法能重载吗？

Dao 接口，就是人们常说的 `Mapper`接口，接口的全限名，就是映射文件中的 namespace 的值，接口的方法名，就是映射文件中`MappedStatement`的 id 值，接口方法内的参数，就是传递给 SQL 的参数。`Mapper`接口是没有实现类的，当调用接口方法时，接口全限名+方法名拼接字符串作为 key 值，可唯一定位一个`MappedStatement`，举例：`com.yaxing.dao.IUserDao.findAll`，可以唯一找到 namespace 为`com.yaxing.dao.IUserDao`下面`id = findAll`的`MappedStatement`。在 MyBatis 中，每一个`<select>`、`<insert>`、`<update>`、`<delete>`标签，都会被解析为一个`MappedStatement`对象，然后执行后续的操作。

```xml
<mapper namespace="com.yaxing.dao.IUserDao">
    <select id="findAll" resultType="com.yaxing.domain.User">
        select * from user;
    </select>
</mapper>
```

Dao 接口里的方法可以重载，但是 MyBatis 的 XML 里面的 ID 不允许重复，也就是说对于同一个接口方法名，mapper 文件中只能有一个该 id。
可以使用动态 SQL 实现接口重载对应的 maper。

```java
/**
 * Mapper接口里面方法重载
 */
public interface StuMapper {
    List<Student> getAllStu();
    List<Student> getAllStu(@Param("id") Integer id);
}
```

然后在 `StuMapper.xml` 中利用 MyBatis 的动态 SQL 就可以实现。

```xml
<select id="getAllStu" resultType="com.pojo.Student">
    select * from student
    <where>
        <if test="id != null">
            id = #{id}
        </if>
    </where>
</select>
```

能正常运行，并能得到相应的结果，这样就实现了在 Dao 接口中写重载方法。

**MyBatis 的 Dao 接口可以有多个重载方法，但是多个接口对应的映射必须只有一个，否则启动会报错。**

### MyBatis是如何将SQL执行结果封装为目标对象并返回的？都有哪些映射形式？

1. 使用 `<resultMap>`标签，逐一定义数据库列名和对象属性名之间的映射关系。

2. 使用 SQL 列的别名功能，将列的别名属性对应为对象属性名。

有了列名与属性名的映射关系后，MyBatis**通过反射创建对象**，同时**使用反射给对象的属性逐一赋值**并返回，那些找不到映射关系的属性，是无法完成赋值的。

### 在mapper中如何传递多个参数?

1. 顺序传参法，\#{} 里面的数字代表传入参数的顺序
2. @Param 注解传参法，\#{} 里面的名称对应的是注解@Param 括号里面修饰的名称
3. map 传参，\#{} 里面的名称对应的是 Map 里面的 key 名称。
4. Java Bean 传参法，\#{} 里面的名称对应的是 User 类里面的成员属性。

### MyBatis动态SQL有什么用？执行原理？有哪些动态SQL？

MyBatis 动态 SQL 可以在 xml 映射文件内，以标签的形式编写动态 SQL.

执行原理是根据表达式的值 完成逻辑判断并动态拼接 SQL。

MyBatis 提供了 9 种动态 SQL 标签： 常用的有 `if|where|foreach`

```xml
<select id="getAllStu" resultType="com.pojo.Student">
    select * from student
    <where>
        <if test="id != null">
            id = #{id}
        </if>
    </where>
</select>
```

### MyBatis实现一对多有几种方式,怎么操作的？

有**联合查询**和**嵌套查询**。

联合查询是几个表联合查询,只查询一次,通过在 resultMap 里面的 collection 节点配置一对多的类就可以完成；

嵌套查询是先查一个表,根据这个表里面的 结果的外键 id,去再另外一个表里面查询数据,也是通过配置 collection,但另外一个表的查询通过 select 节点配置。

### MyBatis是否支持延迟加载？如果支持，它的实现原理是什么？

MyBatis 仅支持 association 关联对象和 collection 关联集合对象的延迟加载。
association 指的就是一对一，collection 指的就是一对多查询。
在 MyBatis 配置文件中，可以配置是否启用延迟加载 lazyLoadingEnabled=true|false。

它的原理是，使用 CGLIB 创建目标对象的代理对象，当调用目标方法时，进入拦截器方法，比如调用 a.getB().getName()，拦截器 invoke()方法发现 a.getB()是 null 值，那么就会单独发送事先保存好的查询关联 B 对象的 SQL，把 B 查询上来，然后调用 a.setB(b)，于是 a 的对象 b 属性就有值了，接着完成 a.getB().getName()方法的调用。这就是延迟加载的基本原理。

### MyBatis的一级、二级缓存

**一级缓存**：一级缓存是 SqlSession 范围的缓存

当我们执行查询之后，查询的结果会同时存入到 SqlSession 为我们提供的一块区域中。该区域的结构是一个 Map。
当我们再次查询同样的数据时，MyBatis 会先去 SqlSession 中查询是否有，有的话直接拿出来用。

当调用 SqlSession 的添加，修改，删除，commit()，close()、clearCache()等方法时，就会清空一级缓存。

> 第一次发起查询用户id为1的用户信息，先去找缓存中是否有id为1的用户信息，如果没有，从数据库查询用户信息。 
> 得到用户信息，将用户信息存储到一级缓存中。 
> 如果SQLSession去执行commit操作（执行插入、更新、删除），清空SqlSession中的一级缓存，这样做的目的为了让缓存中存储的是最新的信息，避免脏读。 
> 第二次发起查询用户id为1的用户信息，先去找缓存中是否有id为1的用户信息，缓存中有，直接从缓存中获取用户信息。

**二级缓存**：二级缓存是 mapper 映射级别的缓存，多个 SqlSession 去操作同一个 Mapper 映射的 SQL 语句，多个 SqlSession 可以共用二级缓存，二级缓存是跨 SqlSession 的。

MyBatis 中 SqlSessionFactory 对象的缓存

​	由同一个 SqlSessionFactory 对象创建的 SqlSession 共享其缓存。

​	二级缓存存放的是数据，而不是对象。

> SQLSession1去查询用户信息，查询到用户信息会将查询数据存储到二级缓存中。
> 如果SqlSession3去执行相同 mapper映射下SQL，执行commit提交，将会清空该 mapper映射下的二级缓存区域的数据。 
> SQLSession2去查询与SQLSession1相同的用户信息，首先会去缓存中找是否存在数据，如果存在直接从缓存中取出数据。

1）一级缓存: 基于 PerpetualCache 的 HashMap 本地缓存，其存储作用域为 Session，当 Session flush 或 close 之后，该 Session 中的所有 Cache 就将清空，默认打开一级缓存；

2）二级缓存与一级缓存其机制相同，默认也是采用 PerpetualCache，HashMap 存储，不同在于其存储作用域为 Mapper(Namespace)，并且可自定义存储源，如 Ehcache。默认不打开二级缓存，要开启二级缓存，使用二级缓存属性类需要实现 Serializable 序列化接口(可用来保存对象的状态),可在它的映射文件中配置 `<cache/>` ；

3）对于缓存数据更新机制，当某一个作用域(一级缓存 Session/二级缓存 Namespaces)的进行了 C/U/D 操作后，默认该作用域下所有 select 中的缓存将被 clear。 

### 什么是MyBatis的接口绑定？有哪些实现方式？

接口绑定，就是在 MyBatis 中任意定义接口,然后把接口里面的方法和 SQL 语句绑定, 我们直接调用接口方法就可以,这样比起原来了 SqlSession 提供的方法我们可以有更加灵活的选择和设置。

接口绑定有两种实现方式：

1. 通过注解绑定，就是在接口的方法上面加上 @Select、@Update 等注解，里面包含 Sql 语句来绑定；
2. 通过 xml 里面写 SQL 来绑定, 在这种情况下,要指定 xml 映射文件里面的 namespace 必须为接口的全路径名。当 Sql 语句比较简单时候,用注解绑定, 当 SQL 语句比较复杂时候,用 xml 绑定,一般用 xml 绑定的比较多。
