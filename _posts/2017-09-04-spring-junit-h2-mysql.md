---
layout: post
title:  "使用h2数据库测试"
date:   2017-09-04 10:58:52
categories: test h2
published: false
comments: true
thread: 20170904101155555
---
使用h2数据库测试
---
## 一、选型
  | testNG  | DBUnit  | 内存数据库
--|---|---|--
优点  | 功能全，可分组测试、依赖测试，综合类等测试	  | 执行数据库单元测试很方便	  |  在内存中模拟数据库，与物理数据库隔离
能否方便的解决数据库现场保留	  | 类似JUnit，TestNG只是测试框架，需要拓展组建支持数据库现场保留	  |  DBUnit是数据库测试框架，需要完成数据库现场保留也需要其他组件	 |  内存数据库的特性能很好的支持数据库现场保留，可以配置DBUnit、JUnit使用

### 1.1 H2内存数据库的优点
  - H2是基于内存的数据库，性能明显好于基于磁盘的数据库
  - 在测试方面，数据库关闭时会清空数据，不会有数据污染
  - 支持索引、全文检索，并且具备多版本控制特性，兼容mysql

### 1.2 H2数据库启动模式
- 嵌入模式（embbed mode）
  - 最简单和快速的模式，在该模式下H2内存数据库随应用启动，且对链接、数据库大小没有限制，缺点是该模式下数据库只能被一个应用所访问，一般只是测试用，问题不大，推荐用嵌入模式启动
- 远程模式（remote mode）
  - 远程模式下，H2数据库启动不受其他应用影响，作为单独的服务存在，可以供多个应用共同访问，但是速度比嵌入模式慢。在H2 server内部H2数据库服务是以嵌入模式启动的
- 混合模式（mixed mode）
  - 混合模式下，本地应用访问H2数据库则和embbed模式，当其他应用想访问该数据库时，表现的和远程模式一样，只是远程模式访问比嵌入式模式访问要慢一些

## 二、配置
### 2.1 pom导入依赖
```xml
<dependency>
   <groupId>com.h2database</groupId>
   <artifactId>h2</artifactId>
   <version>1.4.196</version>  
   <scope>test</scope>
</dependency>
```
推荐用最新版本，一方面性能会更好，另一方面兼容性也会得到提升，老版本的对mysql语法支持的不是很全
### 2.2 结合spring配置H2数据库
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:jdbc="http://www.springframework.org/schema/jdbc"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/jdbc http://www.springframework.org/schema/jdbc/spring-jdbc.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <jdbc:embedded-database id="betaDataSource" type="H2">
       <jdbc:script location="classpath:scripts/H2_TYPE.sql"/>
       <jdbc:script location="classpath:scripts/INIT_TABLE.sql"/>
       <jdbc:script location="classpath:scripts/INIT_DATA.sql"/>
    </jdbc:embedded-database>
    <jdbc:embedded-database id="dataSource" type="H2">
        <jdbc:script location="classpath:scripts/H2_TYPE.sql"/>
        <jdbc:script location="classpath:scripts/INIT_TABLE.sql"/>
        <jdbc:script location="classpath:scripts/INIT_DATA.sql"/>
    </jdbc:embedded-database>
</beans>
```

H2_TYPE.sql ： 配置H2数据库兼容哪种数据库模式
```sql
SET MODE MYSQL;// 设定mode时，还有其他属性可以使用代码配置
```
```java
    mode = new Mode("MySQL");
    mode.convertInsertNullToZero = true;  //转换null 为 0
    mode.indexDefinitionInCreateTable = true;// 允许在创建表语句内部添加索引
    mode.lowerCaseIdentifiers = true;// 元数据消协返回
    mode.onDuplicateKeyUpdate = true;// 插入键值重复时，改为更新操作
    // MySQL allows to use any key for client info entries. See
    // http://grepcode.com/file/repo1.maven.org/maven2/mysql/
    //     mysql-connector-java/5.1.24/com/mysql/jdbc/
    //     JDBC4CommentClientInfoProvider.java
    mode.supportedClientInfoPropertiesRegEx =
            Pattern.compile(".*");
    mode.prohibitEmptyInPredicate = true
```


INIT_TABLE.sql : 设置H2数据库启动时初始化表sql

INIT_DATA.sql : 设置H2数据库启动时初始化数据sql

## 三 注意事项
- 不支持表级注释
```sql
create table tt (
   id int primary key auto_increment
) comment "测试表";
```
- 对同一张表的多次修改操作不能合并成一句sql
- 不支持字段的自动更新
```sql
create table tt (
   id int primary key auto_increment,
   `mod_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '修改时间'
);
```
- H2 unique key 是库级别的
H2中 UNIQUE KEY、KEY是库级别的，而MySQL是表级别的，也就是说在H2中同一个库中不能有两个相同名称的键
