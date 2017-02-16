---
layout: post
title:  "Spring缓存"
date:   2017-01-17 10:58:52
categories: spring cache
published: true
comments: true
thread: 20170117101155555
---
Spring缓存
---

从3.1开始，Spring框架提供了缓存支持。和事务支持类似，提供的Cache Abstract允许接入各种混存实现方案。

`org.springframework.cache.Cache`和`org.springframework.cache.CacheManager`

缓存一个条目就像是`get-if-not-found-then- proceed-and-put-eventually`;多线程试图加载相同条目时，不用有lock应用。
缓存驱逐时逻辑相同，并发访问时有可能拿到旧数据。

使用Cache Abstraction时，开发者需要关注以下两个方面：
- 缓存声明：识别需要缓存的方法及其策略
- 缓存配置：存储和读取数据的缓存


# 基于注解的缓存声明

@Cacheable 被注解的方法触发前，如果有缓存则方法不执行；否则执行方法逻辑
@CacheEvict 能够根据条件清空缓存
@CahePut 每次都会触发真是方法调用，并更新缓存。
@Caching 在一个方法或者类上同时指定多个Spring Cache相关的注解。其拥有三个属性：cacheable、put和evict，分别用于指定@Cacheable、@CachePut和@CacheEvict
@CacheConfig 类级别上共享通用配置

## 默认Key生成策略
因为缓存主要是键值存储，每次对缓存方法的调用需要被翻译成一个合适的缓存获取key。开箱即用原则， Cache Abstraction使用一个简单的 Key生成器基于以下算法：
- 没有方法参数时，返回`SimpleKey.EMPTY`
- 只有一个参数时，返回参数实例
- 多余一个参数时，返回`SimpleKey`包含所遇的参数

上面这种方式大多情况下有效，只要参数有自然key并且实现了合法的`hashCode()`和`equals()`方法。如果不是这样就需要变更key策略了。


# 缓存同步
多线程环境，一些操作可能会并发访问，默认情况下，Cache Abstraction不会加锁并且会执行多次，违背了我们使用缓存的初衷。
对于这种情况, `snyc`属性可以指导基础缓存提供商加锁直至缓存被计算出来，这样只有一个线程会执行方法执行，其他线程会加载缓存；这是一个可选项，要看采用缓存的实现方案是否提供同步支持。

# 启用缓存注解
一种采用注解风格启用

```java
@Configuration
@EnableCaching
public class AppConfig {
}
```

或者使用xml配置

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:cache="http://www.springframework.org/schema/cache"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/cache http://www.springframework.org/schema/cache/spring-cache.xsd">

        <cache:annotation-driven />

</beans>
```
