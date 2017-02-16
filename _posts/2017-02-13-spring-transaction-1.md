---
layout: post
title: Spring 事务
date: 2017-02-13 20:18:52
categories: spring data
published: false
comments: true
thread: 20170213201855556
---

# Spring 事务

### Spring框架事务管理简介
使用Spring框架最令人难以抗拒的理由是 综合的事务支持。
- 一致性的编程模型对于不同的事务API比如JTA、JDBC、Hibernate、JPA和JDO
- 支持声明式事务管理
- 编程性事务管理API逼复杂的事务管理API比如JTA更加简单
- 和Spring data抽象优秀的整合


以下章节叙述Spring框架的事务附加值和技术
- Spring框架的事务支持模型优点
- 理解Spring框架的事务抽象
    - 大体介绍核心class并且描述如何从不同的sources配置获取DataSource实例
- 和事务同步资源
    - 应用代码如何确保资源被正确的created、reuses和cleaned up
- 声明式事务管理
- 编程性事务管理
- 事务边界事件
    - 在事务中如何使用Application Event


### 1. Spring框架的事务支持模型优点

全局事务

本地事务
