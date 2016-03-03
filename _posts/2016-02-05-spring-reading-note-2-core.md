---
layout: post
title:  "Spring源码解析－Spring 核心－IoC"
date:   2016-02-05 03:20:52
categories: spring
published: false
comments: true
thread: 20160205032152555
---

IoC 和 AOP是Spring的核心， 是Spring系统中其他组件模块和应用开发的基础。透过这两个模块的设计和实现可以了解Spring倡导的对企业应用开发所应秉承的思路：
- 易用性。
    POJO开发企业应用, 直接依赖于Java语言，而不是容器和框架。
    提升程序的可测试性，提高软件质量。

- 提供一致性编程模型，面向接口的编程
    降低应用的负载和框架的侵入性。IoC和AOP实现。   
    不作为现有解决方案的替代，而是集成现有。

IoC和AOP这两个核心组件，特别是IoC容器，使用户在使用Spring完成POJO应用开发的过程中必须使用的。这样的应用策略也极大的扩展了Spring的应用场合，不仅包括Java EE应用，还包括其他方面的应用，比如桌面等。从这个意义上讲，IoC容器称得上是Spring的最核心部分。

Agenda：
- 概述
- 设计与实现：BeanFactory
- 初始化过程
- 依赖注入
- 其他相关特性的设计与实现

# 概述
2004年Martin Fowler提出“哪些方面被反转了”，得出的结论“依赖对象的获得被反转了”。控制反转概念在Spring中通过依赖注入实现。

IoC容器，把资源获取的方向反转，让IoC容器主动管理这些依赖关系，将这些依赖关系注入到组件中，让依赖关系的是配合管理更加灵活。

具体的注入实现中，接口注入（Type 1 IoC），setter注入（Type 2 IoC），构造器注入（Type 3 IoC），后两种是主要的；为了防止注入异常，Spring还提供了对特定依赖的检查。

Spring IoC容器已经不是原来简单的[Interface21](http://www.infoq.com/cn/news/2007/05/interface21-gets-funding/)框架了，已经成为一个IoC容器的工业级实现。下面，我们会对IoC容器系列的设计和实现进行详细分析。


# 设计与实现：BeanFactory 和 ApplicationContext
