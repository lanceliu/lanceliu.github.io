---
layout: post
title:  "spring 自动装配集合类型"
date:   2018-10-18 14:11:52
categories: spring
published: true
comments: true
thread: 20181018141155555
---
spring 自动装配集合类型
---
# 问题
有个托管bean中存在这么一个属性
```java
@Autowired private Map<String, Processor> processors;
```
整个context的 xml配置文件和 \*.java文件中排查没有对该属性的初始化。通过调试应用初始化过程发现spring可以对集合类型进行自动装配。

spring 版本3.2.0

AbstractAutowiredCapableBeanFactory.java 的 populateBean 方法中对于后处理器 AutowiredAnotationBeanPostProcessor.java
的inject方法中，会调用 DefaultListableBeanFactory.java 的resolveDependency。
该方法中有对集合型属性进行装备的逻辑描述。

网上有篇详细描述 [自动装备集合属性](https://blog.csdn.net/nlznlz/article/details/82528411)
