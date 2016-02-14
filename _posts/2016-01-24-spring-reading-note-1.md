---
layout: post
title:  "Spring源码解析－读书笔记一－概述"
date:   2016-01-24 20:49:52
categories: spring
comments: true
thread: 20160124204952555
---

## 一、简介
现在一提到Spring，几乎是所有互联初创企业，大部分企业开发的生产标配。问一下你周边的同行“你们开发用的什么框架组合”，回答基本上是“SSH”，“SI”等等。
之所以这么受欢迎，因为简化开发是Spring的主要目标（For the developer, to developer and by the developer）。

Spring Framework 是一个开源的Java/Java EE全功能栈（full-stack）的应用程序框架， 以Apache许可证形式发布，也有.NET平台上的移植版本。
该框架基于Expert One-on-One Java EE Design and Development(ISBN 0-7645-4385-7)一书中的代码，最初由Rod Johnson和Juergen Hoeller等开发。
提供了一个简易的开发方式，这种开发方式，将避免那些可能致使底层代码变得繁杂混乱的大量的属性文件和帮助类。

## 二、Spring特性
Spring框架可以划分为核心、组件和应用三个方面。

我们开发中，Spring在数据持久化、事务处理、消息中间件、分布式计算等方面提供很大方便，并提供了一个基于POJO的轻量级的解决方案.

### 核心部分且其他模块基础：IoC AOP  （反射、代理、字节码）
- POJO的开发模式，复杂的J2EE服务解耦（依赖特定容器）
- 提高单元测试覆盖率

### 组件（用户可以自由选择，Spring的平台开放性）
- 事务处理
- Web MVC
- JDBC
- O/R映射
- 远端调用

### 应用
- DM
- FLEX
- ACEGI

## 三、学习的方式
1. 使用场景
2. 设计和实现过程
3. 源码实现
