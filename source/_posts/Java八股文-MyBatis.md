---
title: Java八股文-MyBatis
copyright: true
mathjax: false
categories:
  - Java八股文
date: 2023-01-23 15:30:00
tags:
toc: true
urlname: java-mybatis
---

> 整理的MyBatis框架相关知识点和面试题，部分内容摘自网络，如有侵权请联系我～<!--more-->


### 什么是MyBatis？

1. MyBatis是一个半ORM（对象关系映射）框架，它内部封装了JDBC，开发时只需要关注SQL语句本身，不需要花费精力去处理**加载驱动、创建连接、创建statement**等繁杂的过程。
2. 通过xml 文件或注解的方式将要执行的各种 statement 配置起来，并通过java对象和 statement中SQL的动态参数进行映射生成最终执行的SQL语句，最后由mybatis框架执行SQL并将结果映射为java对象并返回。（从执行SQL到返回result的过程）。

### ORM是什么

ORM（Object Relational Mapping），对象关系映射，就是把数据库表和实体类及实体类的属性对应起来

让我们可以操作实体类就实现操作数据库表。

### 为什么说MyBatis是半自动ORM映射工具？它与全自动的区别在哪里？

MyBatis在查询关联对象或关联集合对象时，需要手动编写SQL来完成，所以，称之为半自动ORM映射工具。

### JDBC编程有哪些不足之处，MyBatis是如何解决的？

1. 数据库连接创建、释放频繁，造成系统资源浪费从而影响系统性能
2. SQL语句写在代码中造成代码**不易维护**
3. 向SQL语句**传参**麻烦
4. 对**结果集解析**麻烦

### MyBatis编程步骤是什么样的？

1. 读取配置文件，创建SqlSessionFactory工厂
2. 通过SqlSessionFactory工厂创建SqlSession 对象
3. 使用 SqlSession 创建 Dao 接口的代理对象
4. 使用代理对象执行方法
5. 调用session.commit()提交事务
6. 调用session.close()关闭会话

### \#{}和\${}的区别是什么？

mybatis在处理 \#{} 时，会将SQL中的 \#{} 替换为 ? 号，调用PreparedStatement的set方法来赋值。
这是**预编译处理**，可以提高SQL执行效率，更重要的是**可以防止SQL注入**；

mybatis在处理 \${} 时，就是把 \${} 使用**字符串替换**。使用该方式如果传入id的话，会把id=1转换为id="1"字符串形式，而数据库中id实际为数值型这样就不匹配了，查询失败。（\${}是字符串替换，通常在EL表达式中使用，用来展示后端传递到前端的值）。

### 当实体类中的属性名和表中的字段名不一样 ，怎么办 ？

1. 通过在查询的SQL语句中定义数据库字段名的别名，让数据库字段名的别名和实体类的属性名一致。
2. 通过 `<resultMap>`来映射字段名和实体类属性名的一一对应的关系。

### 模糊查询like语句该怎么写?

1. 在Java代码中添加SQL通配符。

	```xml
	string wildcardname = “%smi%”;
	list<name> names = mapper.selectlike(wildcardname);
	
	<select id=”selectlike”>
		select * from foo where bar like #{value}
	</select>
	```

2. 在SQL语句中拼接通配符%，会引起SQL注入

	```xml
	string wildcardname = “smi”;
	list<name> names = mapper.selectlike(wildcardname);
	
	<select id=”selectlike”>
		select * from foo where bar like "%"#{value}"%"
	</select>
	```

### MyBatis都有哪些Executor执行器？它们之间的区别是什么？

MyBatis有三种基本的Executor执行器，SimpleExecutor、ReuseExecutor、BatchExecutor。

- **SimpleExecutor** ：每次开启一个Statement对象，用完立刻关闭Statement对象。
- **ReuseExecutor** ：重复使用创建的Statement对象。
- **BatchExecutor** ：缓存了多个Statement对象，一起执行批量更新。

### 通常一个xml映射文件，都会写一个Dao接口与之对应，请问，这个Dao接口的工作原理是什么？Dao接口里的方法，参数不同时，方法能重载吗？

Dao 接口，就是人们常说的 `Mapper`接口，接口的全限名，就是映射文件中的 namespace 的值，接口的方法名，就是映射文件中`MappedStatement`的 id 值，接口方法内的参数，就是传递给 SQL 的参数。`Mapper`接口是没有实现类的，当调用接口方法时，接口全限名+方法名拼接字符串作为 key 值，可唯一定位一个`MappedStatement`，举例：`com.yaxing.dao.IUserDao.findAll`，可以唯一找到 namespace 为`com.yaxing.dao.IUserDao`下面`id = findAll`的`MappedStatement`。在 MyBatis 中，每一个`<select>`、`<insert>`、`<update>`、`<delete>`标签，都会被解析为一个`MappedStatement`对象，然后执行后续的操作。

```xml
<mapper namespace="com.yaxing.dao.IUserDao">
    <select id="findAll" resultType="com.yaxing.domain.User">
        select * from user;
    </select>
</mapper>
```

Dao 接口里的方法可以重载，但是MyBatis的XML里面的ID不允许重复，也就是说对于同一个接口方法名，mapper文件中只能有一个该id。
可以使用动态SQL实现接口重载对应的maper。

```java
/**
 * Mapper接口里面方法重载
 */
public interface StuMapper {
    List<Student> getAllStu();
    List<Student> getAllStu(@Param("id") Integer id);
}
```

然后在 `StuMapper.xml` 中利用MyBatis的动态SQL就可以实现。

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

能正常运行，并能得到相应的结果，这样就实现了在Dao接口中写重载方法。

**MyBatis 的 Dao 接口可以有多个重载方法，但是多个接口对应的映射必须只有一个，否则启动会报错。**

### MyBatis是如何将SQL执行结果封装为目标对象并返回的？都有哪些映射形式？

1. 使用 `<resultMap>`标签，逐一定义数据库列名和对象属性名之间的映射关系。

2. 使用SQL列的别名功能，将列的别名属性对应为对象属性名。

有了列名与属性名的映射关系后，MyBatis**通过反射创建对象**，同时**使用反射给对象的属性逐一赋值**并返回，那些找不到映射关系的属性，是无法完成赋值的。

### 在mapper中如何传递多个参数?

1. 顺序传参法，\#{} 里面的数字代表传入参数的顺序
2. @Param注解传参法，\#{} 里面的名称对应的是注解@Param括号里面修饰的名称
3. map传参，\#{} 里面的名称对应的是Map里面的key名称。
4. Java Bean传参法，\#{} 里面的名称对应的是User类里面的成员属性。

### MyBatis动态SQL有什么用？执行原理？有哪些动态SQL？

MyBatis动态SQL可以在xml映射文件内，以标签的形式编写动态SQL.

执行原理是根据表达式的值 完成逻辑判断并动态拼接SQL。

MyBatis提供了9种动态SQL标签： 常用的有 `if|where|foreach`

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

联合查询是几个表联合查询,只查询一次,通过在resultMap里面的collection节点配置一对多的类就可以完成；

嵌套查询是先查一个表,根据这个表里面的 结果的外键id,去再另外一个表里面查询数据,也是通过配置collection,但另外一个表的查询通过select节点配置。

### MyBatis是否支持延迟加载？如果支持，它的实现原理是什么？

MyBatis仅支持association关联对象和collection关联集合对象的延迟加载。
association指的就是一对一，collection指的就是一对多查询。
在MyBatis配置文件中，可以配置是否启用延迟加载lazyLoadingEnabled=true|false。

它的原理是，使用CGLIB创建目标对象的代理对象，当调用目标方法时，进入拦截器方法，比如调用a.getB().getName()，拦截器invoke()方法发现a.getB()是null值，那么就会单独发送事先保存好的查询关联B对象的SQL，把B查询上来，然后调用a.setB(b)，于是a的对象b属性就有值了，接着完成a.getB().getName()方法的调用。这就是延迟加载的基本原理。

### MyBatis的一级、二级缓存

**一级缓存**：一级缓存是SqlSession范围的缓存

当我们执行查询之后，查询的结果会同时存入到SqlSession为我们提供的一块区域中。该区域的结构是一个Map。
当我们再次查询同样的数据时，MyBatis会先去SqlSession中查询是否有，有的话直接拿出来用。

当调用SqlSession的添加，修改，删除，commit()，close()、clearCache()等方法时，就会清空一级缓存。

> 第一次发起查询用户id为1的用户信息，先去找缓存中是否有id为1的用户信息，如果没有，从数据库查询用户信息。 
> 得到用户信息，将用户信息存储到一级缓存中。 
> 如果SQLSession去执行commit操作（执行插入、更新、删除），清空SqlSession中的一级缓存，这样做的目的为了让缓存中存储的是最新的信息，避免脏读。 
> 第二次发起查询用户id为1的用户信息，先去找缓存中是否有id为1的用户信息，缓存中有，直接从缓存中获取用户信息。

**二级缓存**：二级缓存是mapper映射级别的缓存，多个SqlSession去操作同一个Mapper映射的SQL语句，多个SqlSession可以共用二级缓存，二级缓存是跨SqlSession的。

MyBatis中SqlSessionFactory对象的缓存

​	由同一个SqlSessionFactory对象创建的SqlSession共享其缓存。

​	二级缓存存放的是数据，而不是对象。

> SQLSession1去查询用户信息，查询到用户信息会将查询数据存储到二级缓存中。
> 如果SqlSession3去执行相同 mapper映射下SQL，执行commit提交，将会清空该 mapper映射下的二级缓存区域的数据。 
> SQLSession2去查询与SQLSession1相同的用户信息，首先会去缓存中找是否存在数据，如果存在直接从缓存中取出数据。

1）一级缓存: 基于 PerpetualCache 的 HashMap 本地缓存，其存储作用域为 Session，当 Session flush 或 close 之后，该 Session 中的所有 Cache 就将清空，默认打开一级缓存；

2）二级缓存与一级缓存其机制相同，默认也是采用 PerpetualCache，HashMap 存储，不同在于其存储作用域为 Mapper(Namespace)，并且可自定义存储源，如 Ehcache。默认不打开二级缓存，要开启二级缓存，使用二级缓存属性类需要实现Serializable序列化接口(可用来保存对象的状态),可在它的映射文件中配置 `<cache/>` ；

3）对于缓存数据更新机制，当某一个作用域(一级缓存 Session/二级缓存Namespaces)的进行了C/U/D 操作后，默认该作用域下所有 select 中的缓存将被 clear。 

### 什么是MyBatis的接口绑定？有哪些实现方式？

接口绑定，就是在MyBatis中任意定义接口,然后把接口里面的方法和SQL语句绑定, 我们直接调用接口方法就可以,这样比起原来了SqlSession提供的方法我们可以有更加灵活的选择和设置。

接口绑定有两种实现方式：

1. 通过注解绑定，就是在接口的方法上面加上 @Select、@Update等注解，里面包含Sql语句来绑定；
2. 通过xml里面写SQL来绑定, 在这种情况下,要指定xml映射文件里面的namespace必须为接口的全路径名。当Sql语句比较简单时候,用注解绑定, 当SQL语句比较复杂时候,用xml绑定,一般用xml绑定的比较多。
