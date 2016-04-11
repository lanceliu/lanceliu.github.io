---
layout: post
title:  "Spring源码解析－Spring 核心－IoC-2"
date:   2016-03-31 23:14:52
categories: spring
published: true
comments: true
thread: 20160331231452555
---

# 四、BeanDefinition IOC的依赖注入
![依赖注入序列](/assets/img/springinternal/dependencyInjection.png)



DefaultListableBeanFactory的基类AbstractBeanFactory的getBean

```Java8
AbstractBeanFactory:

public Object getBean(String name) throws BeansException {
    return doGetBean(name, null, null, false);
}

protected <T> T doGetBean( final String name, final Class<T> requiredType, final Object[] args, boolean typeCheckOnly) {
    ...
    // 从缓存取已经被创建过的
    Object sharedInstance = getSingleton(beanName);
    if (存在) ｛
        // 1: 是普通Bean， 则原路返回
        // 2: 是BeanFactory，则原路返回
        // 3: 是工厂Bean的，根据其工厂生产出该Bean
        bean = getObjectForBeanInstance(sharedInstance, name, beanName, null);


    ｝ else {
        ...
        // 当前BeanFactory找不到则顺着双亲BeanFactory链一直向上着
        ....

        // 获取当前Bean的所有依赖，触发getBean的递归调用。
        String[] dependsOn = mbd.getDependsOn();
        if (dependsOn != null) {
            for (String dependsOnBean : dependsOn) {
                if (isDependent(beanName, dependsOnBean)) {
                    throw new BeanCreationException(mbd.getResourceDescription(), beanName,
                            "Circular depends-on relationship between '" + beanName + "' and '" + dependsOnBean + "'");
                }
                registerDependentBean(dependsOnBean, beanName);
                getBean(dependsOnBean);
            }
        }


        // 创建Bean实例
        if （mbd.isSingleton()） ｛
                    ....
                        return createBean(beanName, mbd, args);
                    .....
        ｝ else if (mbd.isPrototype()) {
            ....
        } else {
            ....
        }
    }

    ....
    return (T) bean;
}
```


getBean是依赖注入的起点，之后会调用createBean，下面通过createBean代码来了解这个实现过程。在这个过程中，Bean对象会依据BeanDefinition定义的要求生成。在AbstractAutowireCapableBeanFactory中实现了createBean

```Java8
AbstractAutowireCapableBeanFactory:

protected Object createBean(String beanName, RootBeanDefinition mbd, Object[] args) throws BeanCreationException {
    ...
    // 检查是否可以实例化，是否可以通过类装载器载入
    Class<?> resolvedClass = resolveBeanClass(mbd, beanName);
    ...
    //如果Bean配置了PostProcessor，那么这里返回的是一个proxy
    Object bean = resolveBeforeInstantiation(beanName, mbdToUse);			
    if (bean != null) {
		return bean;
	}
    ...
    Object beanInstance = doCreateBean(beanName, mbdToUse, args);
    if (logger.isDebugEnabled()) {
        logger.debug("Finished creating instance of bean '" + beanName + "'");
    }
    return beanInstance;
}

// Bean生成的地方
protected Object doCreateBean(final String beanName, final RootBeanDefinition mbd, final Object[] args) {
    // instanceWrapper是用来持有创建出来的bean对象的
    BeanWrapper instanceWrapper = null;
    // 如果是单例，先把缓存中的同名删除
    if (mbd.isSingleton()) {
        instanceWrapper = this.factoryBeanInstanceCache.remove(beanName);
    }
    if (instanceWrapper == null) {
        // 1. 创建bean的地方
        instanceWrapper = createBeanInstance(beanName, mbd, args);
    }
    final Object bean = (instanceWrapper != null ? instanceWrapper.getWrappedInstance() : null);
    Class<?> beanType = (instanceWrapper != null ? instanceWrapper.getWrappedClass() : null);

    // Allow post-processors to modify the merged bean definition.
    synchronized (mbd.postProcessingLock) {
        if (!mbd.postProcessed) {
            applyMergedBeanDefinitionPostProcessors(mbd, beanType, beanName);
            mbd.postProcessed = true;
        }
    }

    ........

    // Initialize the bean instance.
    Object exposedObject = bean;
    try {
        // 2. 对Bean初始化，依赖注入往往发生在这里，
        populateBean(beanName, mbd, instanceWrapper);
        if (exposedObject != null) {
            exposedObject = initializeBean(beanName, exposedObject, mbd);
        }
    }
    .......
}


protected BeanWrapper createBeanInstance(String beanName, RootBeanDefinition mbd, Object[] args) {
........
    // 工厂方法生成
    if (mbd.getFactoryMethodName() != null)  {
        return instantiateUsingFactoryMethod(beanName, mbd, args);
    }

.........
    // 构造函数生成
    Constructor<?>[] ctors = determineConstructorsFromBeanPostProcessors(beanClass, beanName);
    if (ctors != null ||
            mbd.getResolvedAutowireMode() == RootBeanDefinition.AUTOWIRE_CONSTRUCTOR ||
            mbd.hasConstructorArgumentValues() || !ObjectUtils.isEmpty(args))  {
        return autowireConstructor(beanName, mbd, ctors, args);
    }

    // No special handling: simply use no-arg constructor.
    return instantiateBean(beanName, mbd);
}

// 使用默认构造函数实例化
protected BeanWrapper instantiateBean(final String beanName, final RootBeanDefinition mbd) {
...........
            beanInstance = getInstantiationStrategy().instantiate(mbd, beanName, parent);
        }
        BeanWrapper bw = new BeanWrapperImpl(beanInstance);
        initBeanWrapper(bw);
        return bw;

}

SimpleInstantiationStrategy:
// 两种实例化方式
@Override
public Object instantiate(RootBeanDefinition bd, String beanName, BeanFactory owner) {
    // Don't override the class with CGLIB if no overrides.
    if (bd.getMethodOverrides().isEmpty()) {
        .......
        ／/JVM reflection
        return BeanUtils.instantiateClass(constructorToUse);
    }
    else {
        // Must generate CGLIB subclass.
        return instantiateWithMethodInjection(bd, beanName, owner);
    }
}
```


Bean对象创建出来之后，需要依赖关系设置好，主要参考AbstractAutowireCapableBeanFactory的populateBean()
```Java8
protected void populateBean(String beanName, RootBeanDefinition mbd, BeanWrapper bw) {
.......
    // 依赖注入
    if (mbd.getResolvedAutowireMode() == RootBeanDefinition.AUTOWIRE_BY_NAME ||
            mbd.getResolvedAutowireMode() == RootBeanDefinition.AUTOWIRE_BY_TYPE) {
        MutablePropertyValues newPvs = new MutablePropertyValues(pvs);

        // Add property values based on autowire by name if applicable.
        if (mbd.getResolvedAutowireMode() == RootBeanDefinition.AUTOWIRE_BY_NAME) {
            autowireByName(beanName, mbd, bw, newPvs);
        }

        // Add property values based on autowire by type if applicable.
        if (mbd.getResolvedAutowireMode() == RootBeanDefinition.AUTOWIRE_BY_TYPE) {
            autowireByType(beanName, mbd, bw, newPvs);
        }

        pvs = newPvs;
    }
........
    // 属性注入
    applyPropertyValues(beanName, mbd, bw, pvs);
}

protected void applyPropertyValues(String beanName, BeanDefinition mbd, BeanWrapper bw, PropertyValues pvs) {

........
    TypeConverter converter = getCustomTypeConverter();
    if (converter == null) {
        converter = bw;
    }
    // valueResover
    BeanDefinitionValueResolver valueResolver = new BeanDefinitionValueResolver(this, beanName, mbd, converter);

    // Create a deep copy, resolving any references for values.
    List<PropertyValue> deepCopy = new ArrayList<PropertyValue>(original.size());
    boolean resolveNecessary = false;
    // 创建副本
    for (PropertyValue pv : original) {
        ........
                deepCopy.add(new PropertyValue(pv, convertedValue));
        .........
    }
    if (mpvs != null && !resolveNecessary) {
        mpvs.setConverted();
    }

    // Set our (possibly massaged) deep copy.
    try {
        // 依赖注入，会在BeanWrapperImpl中完成
        bw.setPropertyValues(new MutablePropertyValues(deepCopy));
    }
    catch (BeansException ex) {
        throw new BeanCreationException(
                mbd.getResourceDescription(), beanName, "Error setting property values", ex);
    }
}
```
