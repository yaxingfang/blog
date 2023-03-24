---
title: "MySQL 的索引小记"
date: 2023-02-16 13:52:29
categories: ["技术总结"]
tags: ["MySQL"]
url: mysql-index

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

本文对 MySQL 的索引相关操作做简单小记。<!--more-->

## 普通索引

普通索引（由关键字 INDEX 定义的索引）的唯一任务是加快对数据的访问速度。因此，应该只为那些最经常出现在查询条件（WHERE column = …）或排序条件（ORDER BY column）中的数据列创建索引。只要有可能，就应该选择一个数据最整齐、最紧凑的数据列（如一个整数类型的数据列）来创建索引。

```sql
CREATE TABLE mytable(   
	id INT NOT NULL,   
	username VARCHAR(16) NOT NULL,  
	INDEX index_username(username) 
);  

ALTER table mytable ADD INDEX index_username(username);
```

## 唯一索引

普通索引允许被索引的数据列包含重复的值。比如说，因为人有可能同名，所以同一个姓名在同一个”员工个人资料”数据表里可能出现两次或更多次。

如果能确定某个数据列将只包含彼此各不相同的值，在为这个数据列创建索引的时候就应该用关键字 UNIQUE 把它定义为一个唯一索引。这么做的好处：

一是简化了 MySQL 对这个索引的管理工作，这个索引也因此而变得更有效率；

二是 MySQL 会在有新记录插入数据表时，自动检查新记录的这个字段的值是否已经在某个记录的这个字段里出现过了；如果是，MySQL将拒绝插入那条新记录。也就是说，唯一索引可以保证数据记录的唯一性。

事实上，在许多场合，人们创建唯 一索引的目的往往不是为了提高访问速度，而只是为了避免数据出现重复。

```sql
CREATE TABLE mytable(  
	id INT NOT NULL,   
	username VARCHAR(16) NOT NULL,  
	UNIQUE INDEX index_username(username) 
); 

ALTER table mytable ADD UNIQUE INDEX index_username(username); 
```

## 主键索引

一般指主键 id，和唯一索引表达意思相同。

```sql
CREATE TABLE mytable(  
	id INT NOT NULL PRIMARY KEY,   
	username VARCHAR(16) NOT NULL
); 

ALTER TABLE mytable ADD PRIMARY KEY(column);
```

