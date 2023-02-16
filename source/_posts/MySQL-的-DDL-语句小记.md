---
title: MySQL 的 DDL 语句小记
copyright: true
mathjax: false
categories:
  - 技术总结
  - MySQL
toc: false
date: 2023-02-16 13:17:29
tags:
urlname: mysql-ddl
---

本文对 MySQL 的 DDL 语句做简单小记。<!--more-->

**DDL（Data Definition Languages）语句：**数据定义语言，这些语句定义了不同的数据段、数据库、表、列、索引等数据库对象的定义。常用的语句关键字主要包括 create、drop、alter等。

## 建表

MySQL 建表一般设置一些参数：

Field ：字段名
Type：字段类型
Extra：额外（是否 auto_increment)
Default：缺省值
Null ：是否可以为NULL
Key：索引（PRIMARY KEY、UNIQUE INDEX、INDEX)
Comment：备注（MySQL 5.0以上有)

```sql
create table score
(
    id          bigint unsigned auto_increment comment '主键'
        primary key,
    name        varchar(255)                       not null comment '姓名',
    subject     varchar(255)                       not null comment '科目',
    score       int                                not null comment '得分',
    create_time datetime default CURRENT_TIMESTAMP not null comment '创建时间',
    update_time datetime default CURRENT_TIMESTAMP not null on update CURRENT_TIMESTAMP comment '更新时间'
)ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

## 修改表

### 修改字段类型（MODIFY）

**ALTER TABLE 表名 MODIFY [COLUMN] 字段名 字段类型 [FIRST | AFTER col_name];**

例如，修改表 emp 的 ename 字段定义，将 varchar(10)改为 varchar(20)：

```sql
alter table emp modify ename varchar(20); 
```

### 增加字段（ADD）

**ALTER TABLE 表名 ADD [COLUMN] 字段名 字段类型 [FIRST / AFTER col_name];**

例如，表 emp 上新增加字段 age，类型为 int(3)：

```sql
alter table emp add age int(3);
```

### 删除字段（DROP）

**ALTER TABLE 表名 DROP [COLUMN] 字段名;**

例如，将字段 age 删除：

```sql
alter table emp drop age;
```

### 字段改名（CHANGE）

**ALTER TABLE 表名 CHANGE [COLUMN] 字段名 新字段名 字段类型  [FIRST / AFTER col_name];**

例如，将 age 改名为 age1，同时修改字段类型为 int(4)：

```sql
alter table emp change age age1 int(4) ; 
```

注意：change 和 modify 都可以修改表的定义，不同的是 change 后面需要将新列名也要表示出来（就算不修改其名称），不方便。但是 change 的优点是可以修改列名称，modify 则不能。

### 修改字段排列顺序

前面介绍的的字段增加和修改语法（ADD/CNAHGE/MODIFY）中，都有一个可选项 **FIRST / AFTER col_name**，这个选项可以用来修改字段在表中的位置，**默认 ADD 增加的新字段是加在表的最后位置，而 CHANGE/MODIFY 默认都不会改变字段的位置。**

例如，将新增的字段 birth date 加在 ename 之后：

```sql
alter table emp add birth date after ename;
```

修改字段 age，将它放在最前面：

```sql
alter table emp modify age int(3) first;
```

**注意：CHANGE / FIRST / AFTER col_name 这些关键字都属于 MySQL 在标准 SQL 上的扩展，在其他数据库上不一定适用。**

### 表改名（RENAME）

**ALTER TABLE 表名 RENAME 新表名;**

例如，将表 emp 改名为 emp1：

```sql
alter table emp rename emp1
```





