---
layout: post
title:  "mysql rowCount doesn't equal resultSet count"
date:   2016-06-28 09:47:52
categories: mycat mysql
published: true
comments: true
thread: 20160628094752555
---

线上环境新版本发布后，使用java 的mysql驱动查询总条数总是100，但库内总数是500多。
使用mySqlWorkBench查询是正确的，但使用java的mysql驱动查询总是100条？
    1. 首先排除驱动问题，因为发布之前是OK的。
    2. 客户端配置缘故,查询[Mysql参数列表](https://dev.mysql.com/doc/connector-j/5.1/en/connector-j-reference-configuration-properties.html)没有发现该配置；
    3. 只能是该新版本发布时中间件配置的缘故，在schema.xml 有一项sqlMaxLimit配置。该属性相当于 select语句中的limit默认配置，如果发送到中间件的查询语句没有加limit时，会使用sqlMaxLimit的属性值。
