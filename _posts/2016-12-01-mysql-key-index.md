---
layout: post
title:  "mysql中key 、primary key 、unique key 与index区别"
date:   2016-12-01 09:50:52
categories: database
published: true
comments: true
thread: 20161201095055555
---
mysql中key 、primary key 、unique key 与index区别
---

1. key 是数据库的物理结构，它包含两层意义
一是约束（偏重于约束和规范数据库的结构完整性）
二是索引（辅助查询用的）

primary key 有两个作用，一是约束作用（constraint），用来规范一个存储主键和唯一性，但同时也在此key上建立了一个index；
     一个表只能有一个 primary key
     必须not null
     数据不能重复

sql create: PRIMARY KEY(id)
sql append: ALTER TABLE [table] ADD PRIMARY KEY ([Id]);
sql delete: ALTER TABLE [table] DROP PRIMARY KEY;

unique key  也有两个作用，一是约束作用（constraint），规范数据的唯一性，但同时也在这个key上建立了一个index；
    可以有多个unique key
    没有 not null约束
    数据不能重复

sql create:
    - UNIQUE KEY [unique_key_name] (col1, col2) # in create table statements
    - UNIQUE (col1, col2) # in create table statements
    - CONSTRAINT [unique_key_name] UNIQUE (Id_P,LastName)
sql append:
    - ALTER TABLE [table_name] ADD CONSTRAINT [unique_key_name] UNIQUE (Id_P,LastName)
    - alter table [table_name] add unique key [unique_key_name] (`col1`,`col2`);
sql delete: ALTER TABLE [table] DROP INDEX [unique_key_name]

foreign key 也有两个作用，一是约束作用（constraint），规范数据的引用完整性，但同时也在这个key上建立了一个index；

2. index是数据库的物理结构，它只是辅助查询的.它创建时会在另外的表空间（mysql中的innodb表空间）以一个类似目录的结构存储
