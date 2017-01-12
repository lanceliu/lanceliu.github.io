---
layout: post
title:  "Spring源码解析－Spring 核心－IoC-3"
date:   2016-04-11 20:50:52
categories: spring
published: false
comments: true
thread: 20160411205052555
---

# 五、容器的其它相关特性的设计和实现
## 1. ApplicationContext和Bean的初始化及销毁

## 2. lazy-init属性和预实例化

## 3. FactoryBean的实现

## 4. BeanPostProcessor的实现
[参考](http://jinnianshilongnian.iteye.com/blog/1492424)
- InstantiationAwareBeanPostProcessor
    - postProcessBeforeInstantiation：在实例化目标对象之前执行，可以自定义实例化逻辑，如返回一个代理对象等。如果此处返回的Bean不为null将中断后续Spring创建Bean的流程，且只执行postProcessAfterInitialization回调方法
    - postProcessAfterInstantiation：Bean实例化完毕后执行的后处理操作，所有初始化逻辑、装配逻辑之前执行，如果返回false将阻止其他的InstantiationAwareBeanPostProcessor的postProcessAfterInstantiation的执行.
    - postProcessPropertyValues ：完成其他定制的一些依赖注入和依赖检查等，在属性apply之前。如AutowiredAnnotationBeanPostProcessor执行@Autowired注解注入，CommonAnnotationBeanPostProcessor执行@Resource等注解的注入，PersistenceAnnotationBeanPostProcessor执行@ PersistenceContext等JPA注解的注入，RequiredAnnotationBeanPostProcessor执行@ Required注解的检查等等，
- MergedBeanDefinitionPostProcessor
- SmartInstantiationAwareBeanPostProcessor
- BeanPostProcessor
    - postProcessBeforeInitialization：实例化、依赖注入完毕，在调用显示的初始化之前完成一些定制的初始化任务，如BeanValidationPostProcessor完成JSR-303 @Valid注解Bean验证，InitDestroyAnnotationBeanPostProcessor完成@PostConstruct注解的初始化方法调用，ApplicationContextAwareProcessor完成一些Aware接口的注入（如EnvironmentAware、ResourceLoaderAware、ApplicationContextAware），其返回值将替代原始的Bean对象
    - postProcessAfterInitialization：实例化、依赖注入、初始化完毕时执行，如AspectJAwareAdvisorAutoProxyCreator（完成xml风格的AOP配置(<aop:config>)的目标对象包装到AOP代理对象）、AnnotationAwareAspectJAutoProxyCreator（完成@Aspectj注解风格（<aop:aspectj-autoproxy> @Aspect）的AOP配置的目标对象包装到AOP代理对象），其返回值将替代原始的Bean对象；
- DestructionAwareBeanPostProcessor

## 5. autowiring(自动依赖装配)的实现

## 6. bean的依赖检查

## 7. Bean对IoC容器的感知


Synthetic： class or method 是通过compiler生成的。 比如在代码中创建的匿名内部类。
