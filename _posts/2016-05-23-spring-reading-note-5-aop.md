---
layout: post
title:  "Spring AOP"
date:   2016-05-23 09:47:52
categories: spring
published: true
comments: true
thread: 20160523094752555
---

AOP 是 Aspect-Oriented-Programming, 是一种新的模块化机制，链接散落在对象，类或函数中的链接点，从业务逻辑中抽取出解决特定领域问题的代码，便于维护。

Proxy代理对象和拦截器字节码翻译技术， 来实现切面应用的各种编织实现和环绕增强。

AOP联盟对AOP体系结构做出了定义， 该结构涉及的概念由高到低大致分为三个层次（从高到低）：
- 语言和开发环境
    - 基础：待增强的对象
    - 切面：对于基础的增强
    - 配置：可以看成一种变质，把基础和切面结合起来
- 面向方面的系统： 配置和编织的实现
- 底层编织实现模块
    - 反射
    - 程序预处理
    - 拦截器框架
    - 类装载器框架
    - 元数据处理

# AOP概述
## 通知
在jointPoint出做什么，为aspect增强提供织入接口。
具体定义 org.aopalliance.aop.Advice
具体的通知类型： BeforeAdvice、AfterAdvice、ThrowsAdvice

## 切点
Advice应该作用于哪些连接点，被作用的jointPoint的集合。
```java
public interface Pointcut {

	/**
	 * Return the ClassFilter for this pointcut.
	 * @return the ClassFilter (never {@code null})
	 */
	ClassFilter getClassFilter();

	/**
	 * Return the MethodMatcher for this pointcut.
	 * @return the MethodMatcher (never {@code null})
	 */
	MethodMatcher getMethodMatcher();


	/**
	 * Canonical Pointcut instance that always matches.
	 */
	Pointcut TRUE = TruePointcut.INSTANCE;

}
```

## 通知器
Advisor 完成对目标方法的增强的结合。
以Spring的DefaultPointcutAdvisor为例来了解Advisor的工作原理
```java
public class DefaultPointcutAdvisor extends AbstractGenericPointcutAdvisor implements Serializable {

    // 默认匹配所有的方法
	private Pointcut pointcut = Pointcut.TRUE;

	/**
	 * 创建一个空对象，默认Pointcut.TRUE.
	 * 使用这个对象之前必须设定Advice
	 */
	public DefaultPointcutAdvisor() {
	}

	public DefaultPointcutAdvisor(Advice advice) {
		this(Pointcut.TRUE, advice);
	}
	public DefaultPointcutAdvisor(Pointcut pointcut, Advice advice) {
		this.pointcut = pointcut;
		setAdvice(advice);
	}

	public void setPointcut(Pointcut pointcut) {
		this.pointcut = (pointcut != null ? pointcut : Pointcut.TRUE);
	}

	@Override
	public Pointcut getPointcut() {
		return this.pointcut;
	}

	@Override
	public String toString() {
		return getClass().getName() + ": pointcut [" + getPointcut() + "]; advice [" + getAdvice() + "]";
	}
}
```

# 建立AopProxy代理对象
## 设计原理
AOP模块，一个主要部分是代理对象的生成；
- Spring应用，通过配置和调用Spring的ProxyFactoryBean来完成。 可以使用Proxy和CGLIB两种方式。

以ProxyFactory的设计为中心，相应的继承关系如下：

- ProxyConfig 提供配置相关属性.
    - AdvisedSupport 封装了AOP对通知和通知器的操作
        - ProxyCreatorSupport 创建AOP对象的一个辅助类
            - AspectJProxyFactory 集成spring和aspectJ作用
            - ProxyFactory 编程式AOP
            - ProxyFactoryBean 容器内声明式AOP
        - AbstractSingletonProxyFactoryBean 为FactoryBean的Object类型生成的单例代理
            - TransactionProxyFactoryBean

ProxyConfig:
```java
public class ProxyConfig implements Serializable {
    /**
        设置是否直接代理的目标类，而不是只代理特定的接口。
    */
	private boolean proxyTargetClass = false;
    /**
    是否做一些激进的优化。比如通知发生变化后也不会生效在代理被创建以后
    */
	private boolean optimize = false;

    /**
    不透明。
    */
	boolean opaque = false;

	boolean exposeProxy = false;

	private boolean frozen = false;
```

# ProxyFactoryBean配置
