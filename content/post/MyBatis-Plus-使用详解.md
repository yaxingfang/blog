---
title: "MyBatis-Plus 使用详解"
date: 2023-02-27 16:08:48
categories: ["技术总结"]
tags: ["MyBatis"]
url: mybatis-plus

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

本文对 MyBatis-Plus 的使用做简单小记。<!--more-->

## 入门

### 建表

```sql
DROP TABLE IF EXISTS user;

CREATE TABLE user (
  id BIGINT(20) PRIMARY KEY NOT NULL COMMENT '主键',
  name VARCHAR(30) DEFAULT NULL COMMENT '姓名',
  age INT(11) DEFAULT NULL COMMENT '年龄',
  email VARCHAR(50) DEFAULT NULL COMMENT '邮箱',
  manager_id BIGINT(20) DEFAULT NULL COMMENT '直属上级id',
  create_time DATETIME DEFAULT NULL COMMENT '创建时间'
) ENGINE=INNODB CHARSET=UTF8;

INSERT INTO user (id, name, age ,email, manager_id, create_time) VALUES
(1, '大BOSS', 40, 'boss@baomidou.com', NULL, '2021-03-22 09:48:00'),
(2, '李经理', 40, 'boss@baomidou.com', 1, '2021-01-22 09:48:00'),
(3, '黄主管', 40, 'boss@baomidou.com', 2, '2021-01-22 09:48:00'),
(4, '吴组长', 40, 'boss@baomidou.com', 2, '2021-02-22 09:48:00'),
(5, '小菜', 40, 'boss@baomidou.com', 2, '2021-02-22 09:48:00')
```

### 引入依赖，设置配置文件

引入依赖 pom.xml

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
        <scope>test</scope>
    </dependency>

    <dependency>
        <groupId>com.baomidou</groupId>
        <artifactId>mybatis-plus-boot-starter</artifactId>
    </dependency>

    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
    </dependency>

    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
    </dependency>
</dependencies>
```

配置文件 `application.yaml`

```yaml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/java?serverTimezone=Asia/Shanghai
    username: root
    password: FYX123fyx.

# 开启SQL语句打印
mybatis-plus:
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
```

### 生成实体类和 Mapper

实体类 `com.yaxing.entity.User`

```java
@TableName(value = "user")
@Data
public class User implements Serializable {
    /**
     * 主键
     */
    @TableId(value = "id")
    private Long id;

    /**
     * 姓名
     */
    @TableField(value = "name")
    private String name;

    /**
     * 年龄
     */
    @TableField(value = "age")
    private Integer age;

    /**
     * 邮箱
     */
    @TableField(value = "email")
    private String email;

    /**
     * 直属上级id
     */
    @TableField(value = "manager_id")
    private Long managerId;

    /**
     * 创建时间
     */
    @TableField(value = "create_time")
    private Date createTime;

    @TableField(exist = false)
    private static final long serialVersionUID = 1L;
}
```

mapper 文件 `com.yaxing.mapper.UserMapper`

```java
@Mapper
public interface UserMapper extends BaseMapper<User> {
}
```

### 主启动类添加注解扫描

```java
@SpringBootApplication
@MapperScan("com.yaxing.mapper")
public class MybatisPlusDemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(MybatisPlusDemoApplication.class, args);
    }

}
```

### 测试

```java
@RunWith(SpringRunner.class)
@SpringBootTest(classes = MybatisPlusDemoApplication.class)
public class MybatisPlusDemoApplicationTest {
    @Autowired
    private UserMapper userMapper;

    @Test
    public void testSelect() {
        List<User> list = userMapper.selectList(null);
        Assert.assertEquals(5, list.size());
        list.forEach(System.out::println);
    }
}
```

测试结果

```sh
==>  Preparing: SELECT id,name,age,email,manager_id,create_time FROM user
==> Parameters: 
<==    Columns: id, name, age, email, manager_id, create_time
<==        Row: 1, 大BOSS, 40, boss@baomidou.com, null, 2021-03-22 09:48:00
<==        Row: 2, 李经理, 40, boss@baomidou.com, 1, 2021-01-22 09:48:00
<==        Row: 3, 黄主管, 40, boss@baomidou.com, 2, 2021-01-22 09:48:00
<==        Row: 4, 吴组长, 40, boss@baomidou.com, 2, 2021-02-22 09:48:00
<==        Row: 5, 小菜, 40, boss@baomidou.com, 2, 2021-02-22 09:48:00
<==      Total: 5
Closing non transactional SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@4cbc2e3b]
User(id=1, name=大BOSS, age=40, email=boss@baomidou.com, managerId=null, createTime=Mon Mar 22 09:48:00 CST 2021)
User(id=2, name=李经理, age=40, email=boss@baomidou.com, managerId=1, createTime=Fri Jan 22 09:48:00 CST 2021)
User(id=3, name=黄主管, age=40, email=boss@baomidou.com, managerId=2, createTime=Fri Jan 22 09:48:00 CST 2021)
User(id=4, name=吴组长, age=40, email=boss@baomidou.com, managerId=2, createTime=Mon Feb 22 09:48:00 CST 2021)
User(id=5, name=小菜, age=40, email=boss@baomidou.com, managerId=2, createTime=Mon Feb 22 09:48:00 CST 2021)
```

## 注解

### @TableName

- 描述：表名注解，标识实体类对应的表
- 使用位置：实体类

### @TableId

- 描述：主键注解
- 使用位置：实体类主键字段

### @TableField

- 描述：字段注解（非主键）

## Mapper CRUD 接口

说明:

- 通用 CRUD 封装 BaseMapper 接口，为 `Mybatis-Plus` 启动时自动解析实体表关系映射转换为 `Mybatis` 内部对象注入容器
- 泛型 `T` 为任意实体对象
- 参数 `Serializable` 为任意类型主键 `Mybatis-Plus` 不推荐使用复合主键约定每一张表都有自己的唯一 `id` 主键
- 对象 `Wrapper` 为 条件构造器

### Insert

```java
// 插入一条记录
int insert(T entity);
```

| 类型 | 参数名 |   描述   |
| :--: | :----: | :------: |
|  T   | entity | 实体对象 |

### Delete

```java
// 根据 entity 条件，删除记录
int delete(@Param(Constants.WRAPPER) Wrapper<T> wrapper);
// 删除（根据ID 批量删除）
int deleteBatchIds(@Param(Constants.COLLECTION) Collection<? extends Serializable> idList);
// 根据 ID 删除
int deleteById(Serializable id);
// 根据 columnMap 条件，删除记录
int deleteByMap(@Param(Constants.COLUMN_MAP) Map<String, Object> columnMap);
```

|                类型                |  参数名   |                 描述                 |
| :--------------------------------: | :-------: | :----------------------------------: |
|            Wrapper\<T\>            |  wrapper  |  实体对象封装操作类（可以为 null）   |
| Collection<? extends Serializable> |  idList   | 主键 ID 列表(不能为 null 以及 empty) |
|            Serializable            |    id     |               主键 ID                |
|        Map<String, Object>         | columnMap |           表字段 map 对象            |

### Update

```java
// 根据 whereWrapper 条件，更新记录
int update(@Param(Constants.ENTITY) T updateEntity, @Param(Constants.WRAPPER) Wrapper<T> whereWrapper);
// 根据 ID 修改
int updateById(@Param(Constants.ENTITY) T entity);
```

|     类型     |    参数名     |                             描述                             |
| :----------: | :-----------: | :----------------------------------------------------------: |
|      T       |    entity     |               实体对象 (set 条件值,可为 null)                |
| Wrapper\<T\> | updateWrapper | 实体对象封装操作类（可以为 null,里面的 entity 用于生成 where 语句） |

### Select

```java
// 根据 ID 查询
T selectById(Serializable id);
// 根据 entity 条件，查询一条记录
T selectOne(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);

// 查询（根据ID 批量查询）
List<T> selectBatchIds(@Param(Constants.COLLECTION) Collection<? extends Serializable> idList);
// 根据 entity 条件，查询全部记录
List<T> selectList(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
// 查询（根据 columnMap 条件）
List<T> selectByMap(@Param(Constants.COLUMN_MAP) Map<String, Object> columnMap);
// 根据 Wrapper 条件，查询全部记录
List<Map<String, Object>> selectMaps(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
// 根据 Wrapper 条件，查询全部记录。注意： 只返回第一个字段的值
List<Object> selectObjs(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);

// 根据 entity 条件，查询全部记录（并翻页）
IPage<T> selectPage(IPage<T> page, @Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
// 根据 Wrapper 条件，查询全部记录（并翻页）
IPage<Map<String, Object>> selectMapsPage(IPage<T> page, @Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
// 根据 Wrapper 条件，查询总记录数
Integer selectCount(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
```

|                类型                |    参数名    |                   描述                   |
| :--------------------------------: | :----------: | :--------------------------------------: |
|            Serializable            |      id      |                 主键 ID                  |
|            Wrapper\<T\>            | queryWrapper |    实体对象封装操作类（可以为 null）     |
| Collection<? extends Serializable> |    idList    |   主键 ID 列表(不能为 null 以及 empty)   |
|        Map<String, Object>         |  columnMap   |             表字段 map 对象              |
|             IPage\<T\>             |     page     | 分页查询条件（可以为 RowBounds.DEFAULT） |

## 逻辑删除

### 说明

只对自动注入的 sql 起效：

- 插入：不作限制
- 查找：追加 where 条件过滤掉已删除数据,如果使用 wrapper.entity 生成的 where 条件也会自动追加该字段
- 更新：追加 where 条件防止更新到已删除数据,如果使用 wrapper.entity 生成的 where 条件也会自动追加该字段
- 删除：转变为 更新

例如：

- 删除：`update user set deleted=1 where id = 1 and deleted=0`
- 查找：`select id,name,deleted from user where deleted=0`

字段类型支持说明：

- 支持所有数据类型(推荐使用 `Integer`,`Boolean`,`LocalDateTime`)
- 如果数据库字段使用`datetime`,逻辑未删除值和已删除值支持配置为字符串`null`,另一个值支持配置为函数来获取值如`now()`

附录：

- 逻辑删除是为了方便数据恢复和保护数据本身价值等等的一种方案，但实际就是删除。
- 如果你需要频繁查出来看就不应使用逻辑删除，而是以一个状态去表示。

### 使用方法

1、配置 `application.yaml`

```yaml
mybatis-plus:
  global-config:
    db-config:
      logic-delete-field: flag # 全局逻辑删除的实体字段名(since 3.3.0,配置后可以忽略不配置步骤2)
      logic-delete-value: 1 # 逻辑已删除值(默认为 1)
      logic-not-delete-value: 0 # 逻辑未删除值(默认为 0)
```

2、实体类字段上加上`@TableLogic`注解

```java
@TableLogic
private Integer deleted;
```

前面在 `application.yaml` 中的配置是全局的。通常来说，对于多个表，我们也会统一逻辑删除字段的名称，统一逻辑已删除和未删除的值，所以全局配置即可。当然，若要对某些表进行单独配置，在实体类的对应字段上使用`@TableLogic`即可。

```java
@TableLogic(value = "0", delval = "1")
private Integer deleted;
```

### 常见问题

**如何 insert ?**

1. 字段在数据库定义默认值(推荐)
2. insert 前自己 set 值
3. 使用 [自动填充功能](https://baomidou.com/pages/4c6bcf/)

**删除接口自动填充功能失效**

1. 使用 `deleteById` 方法(推荐)
2. 使用 `update` 方法并: `UpdateWrapper.set(column, value)`(推荐)
3. 使用 `update` 方法并: `UpdateWrapper.setSql("column=value")`
4. 使用 [Sql 注入器](https://baomidou.com/pages/42ea4a/) 注入 `com.baomidou.mybatisplus.extension.injector.methods.LogicDeleteByIdWithFill` 并使用(3.5.0版本已废弃，推荐使用deleteById)

## 参考

https://baomidou.com/

https://juejin.cn/post/6961721367846715428
