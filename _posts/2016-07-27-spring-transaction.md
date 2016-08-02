---
layout: post
title:  "Spring 事务"
date:   2016-07-27 09:47:52
categories: spring
published: false
comments: true
thread: 20160727094752555
---

# 事务处理
# 事务处理设计概览
# 事务处理应用场景
# 声明式事务处理
# 事务处理的设计与实现
# 事务处理器的设计与实现

![class hierarchy](/assets/img/springinternal/transaction_class_hierarchy.png)

TransactionProxyFactoryBean 实现AOP功能， 生成Proxy代理。

代理对象通过TransactionInterceptor完成对代理方法的拦截。
