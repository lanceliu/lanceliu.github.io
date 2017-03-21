---
layout: post
title:  "Spring源码解析－Spring 核心－IoC"
date:   2016-02-05 03:20:52
categories: spring
published: true
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

# 一、概述
2004年Martin Fowler提出“哪些方面被反转了”，得出的结论“依赖对象的获得被反转了”。控制反转概念在Spring中通过依赖注入实现。

IoC容器，把资源获取的方向反转，让IoC容器主动管理这些依赖关系，将这些依赖关系注入到组件中，让依赖关系的是配合管理更加灵活。

具体的注入实现中，接口注入（Type 1 IoC），setter注入（Type 2 IoC），构造器注入（Type 3 IoC），后两种是主要的；为了防止注入异常，Spring还提供了对特定依赖的检查。

Spring IoC容器已经不是原来简单的[Interface21](http://www.infoq.com/cn/news/2007/05/interface21-gets-funding/)框架了，已经成为一个IoC容器的工业级实现。下面，我们会对IoC容器系列的设计和实现进行详细分析。


# 二、设计与实现：BeanFactory 和 ApplicationContext
IoC容器主要系列：
一个是实现BeanFactory接口的简单容器系列，这系列之实现了容器的最基本功能；
另一个是ApplicationContext应用上下文，作为容器的高级形态而存在。


BeanDefinition来管理对象以及之间的相互依赖关系：
- 抽象了对Bean的定义，是让容器起作用的主要数据类型；
- 对象依赖关系的数据抽象

BeanDefinition像是容器里的水，BeanFactory是容器。

### Spring容器系列如下图：

![容器种类](/assets/img/springinternal/springIocContainer.png)

### IoC容器具体的功能，主要体现在BeanFactory接口：
![容器设计](/assets/img/springinternal/beanFactoryDesign.png)

1. BeanFactory 到 HierarchicalBeanFactory 再到ConfigureBeanFactory，是一条主要的设计路径.
    HierarchicalBeanFactory定义了getparentBeanFactory的功能，具备了双亲IoC容器的管理功能；
    ConfigureBeanFactory增加了一些配置功能，比如setParentBeanFactory设置双亲容器，addBeanPostProcessor配置后置处理器；

2. 第二条接口设计主线是，以ApplicationContext应用上下文接口为核心的接口设计。
    ListableBeanFactory细化了许多BeanFactory功能；


## BeanFactory应用场景
- 定义了IoC容器最基本的形式，应该遵守的最基本的服务契约
- 使用容器，可以使用转义符"&"得道FactoryBean本身，而不是这个FactoryBean生产的bean对象

## BeanFactory设计原理
以XmlBeanFactory实现为例说明简单IoC容器的设计原理。下图为XmlBeanFactory设计的类继承关系：
![XmlBeanFactory类图](/assets/img/springinternal/XmlBeanFactory.png)

- XmlBeanFactory是Rod Johnson在2001年时就写下的代码，该类继承自DefaultListableBeanFactory,后者十分重要，包含了几本IoC容器所具有的基本功能。
- Xml定义信息的处理不是XmlBeanFactory直接完成的。在内部初始化了XmlBeanDefinitionReader

编程式使用IoC容器：

```Java8
ClassPathResource res = new ClassPathResource("first.xml");
DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(factory);
reader.loadBeanDefinitions(res);
HelloBean helloBean = (HelloBean) factory.getBean("hello")；
```

1. 创建IoC配置文件抽象资源，包含了BeanDefinition
2. 创建一个BeanFactory，使用DefaultListableBeanFactory
3. 创建一个BeanDefinition读取器，读取完毕后回调配置给BeanFactory
4. 从第一步配置好的资源位置读取配置信息，由XmlBeanDefinitionReader完成。完成整个载入和注册Bean定义后，就可以直接使用IoC容器了

## ApplicationContext应用场景
在BeanFactory的基础上添加的附加功能:
- 支持不同的信息源，阔赞了MessageSource接口，支持国际化
- 访问资源。对ResourceLoader和Resource的支持，可以从不同地方获取Bean定义资源
- 支持应用事件。引入事件机制。
- 建议使用ApplicationContext在开发应用时。

## ApplicationContext设计原理
以FileSystemXmlApplicationContext为例说明设计原理，FileSystemXmlApplicationContext的基类AbstractXmlApplicationContext中实现了，作为一个具体的应用上下文，只需要实现和他自身设计相关的两个功能：
- 如果直接使用FileSystemXmlApplicationContext，对于实例化这个应用上下文的支持，同时启动IoC容器的refresh过程

```Java8
public FileSystemXmlApplicationContext(String[] configLocations, boolean refresh, ApplicationContext parent) throws BeansException {
    super(parent);
    setConfigLocations(configLocations);
    if (refresh) {
        refresh();
    }
}
```

- 与FileSystemXmlApplicationContext设计具体相关的功能，这部分与怎样从文件系统加载Xml的bean定义资源有关

```Java8
    protected Resource getResourceByPath(String path) {
        if (path != null && path.startsWith("/")) {
            path = path.substring(1);
        }
        return new FileSystemResource(path);
    }
```

# 三、IoC容器的初始化
以下是AbstractApplicationContext.refresh()方法的具体内容

```Java8

		// Prepare this context for refreshing.
		prepareRefresh();

		// Tell the subclass to refresh the internal bean factory.
		ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

		// Prepare the bean factory for use in this context.
		prepareBeanFactory(beanFactory);

		try {
			// Allows post-processing of the bean factory in context subclasses.
			postProcessBeanFactory(beanFactory);

			// Invoke factory processors registered as beans in the context.
			invokeBeanFactoryPostProcessors(beanFactory);

			// Register bean processors that intercept bean creation.
			registerBeanPostProcessors(beanFactory);

			// Initialize message source for this context.
			initMessageSource();

			// Initialize event multicaster for this context.
			initApplicationEventMulticaster();

			// Initialize other special beans in specific context subclasses.
			onRefresh();

			// Check for listener beans and register them.
			registerListeners();

			// Instantiate all remaining (non-lazy-init) singletons.
			finishBeanFactoryInitialization(beanFactory);

			// Last step: publish corresponding event.
			finishRefresh();
		}

		catch (BeansException ex) {
			if (logger.isWarnEnabled()) {
				logger.warn("Exception encountered during context initialization - " +
						"cancelling refresh attempt: " + ex);
			}

			// Destroy already created singletons to avoid dangling resources.
			destroyBeans();

			// Reset 'active' flag.
			cancelRefresh(ex);

			// Propagate exception to caller.
			throw ex;
		}

		finally {
			// Reset common introspection caches in Spring's core, since we
			// might not ever need metadata for singleton beans anymore...
			resetCommonCaches();
		}
	}
```
IoC容器的初始化是由refresh方法来完成启动的。具体来讲，这个启动包括BeanDefinition的Resource定位、载入和注册。
- 第一个过程Resource 定位。由ResourceLoader通过统一的Resource接口完成。
- 第二个过程是BeanDefinition的载入。一般使用BeanDefinitionReader载入。
- 第三个过程是想IoC容器注册这些BeanDefinition过程。通过调用BeanDefinitionRegistry接口来完成。在内部将BeanDefinition注入到一个HashMap中去，
IoC容器就是通过这个HashMap来持有这些BeanDefinition的。
初始化过程中，一般是不包含依赖注入的，Bean的载入和注入是两个独立的过程，注入一般发生在第一次通过getBean想容器索取Bean的时候。


### BeanDefinition 的Resource定位
```Java8
public FileSystemXmlApplicationContext(String[] configLocations, boolean refresh, ApplicationContext parent)
        throws BeansException {

    // 1 获取资源加载器；资源路径解析；系统属性设置
    super(parent);
    // 2 设定加载位置；解析加载路径
    setConfigLocations(configLocations);
    if (refresh) {
        // 3 载入BeanDefinition，并注册
        refresh();
    }
}
```
在步骤1，2完成资源的定位


### BeanDefinition的载入和解析
针对以上例子，得出Spring解析xml文件的流程图和结构图：

![加载时许图](/assets/img/springinternal/ioc-loadresource-seq.jpg)
    ![SpringReader架构](/assets/img/springinternal/spring-reader-structure.jpg)

承担载入和解析任务为refresh方法中的obtainFreshBeanFactory方法。
AbstractXmlApplicationContext.loadBeanDefinitions来载入BeanDefinition。

Spring解析配置文件中用到的一些关键类的介绍：

- Resource：各种资源的抽象接口，包括xml文件，网络上的资源等。
- BeanDefinitionRegistry：用于注册BeanDefinitionRegistry。
- BeanDefinitionReader：用于读取解析Resource的抽象接口。
- DefaultBeanDefinitionDocumentReader：实现了BeanDefinitionDocumentReader接口，DefaultBeanDefinitionDocumentReader并不负责任何具体的bean解析，它面向的是xml Document对象，根据其元素的命名空间和名称，起一个类似路由的作用（（不过，命名空间的判断，也是委托给delegate来做的），它跟BeanDefinitionParserDelegate协同合作，把解析任务交接BeanDefinitionParserDelegate来做。
- BeanDefinitionParserDelegate：完成具体Bean的解析(比如<bean>、<import>、<alias>标签)，对于扩展的标签会交给不同的NamespaceHandler跟BeanDefinitionParser来解析。
- BeanDefinitionParser：解析配置文件成相应的BeanDefinition(<context:component-scan>,<aop:config>等标签都是又不同的BeanDefinitionParser来解析)，一般在NamespaceHandler中使用。Spring也为自定义BeanDefinitionParser提供了很多支持，在一些抽象类的基础上添加少量功能即可满足大部分需求。(@Configuration,@Bean 都是这么加载的)
- NamespaceHandler：要解析自定义的bean就要通过自己所实现的NamespaceHandler来进行解析。比如定义了http\://www.springframework.org/schema/osgi=org.springframework.osgi.config.OsgiNamespaceHandler,那么在碰到osgi的scheme的时候就会去调用OsgiNamespaceHandler来进行解析； 在对于普通的扩展需求来说，只要让自己的Handler继承NamespaceHandlerSupport并实现 init()方法 就好了，对于特殊的扩展需求 则可以自己 来实现NamespaceHandler。

**源码一：加载XML**
```Java8
AbstractXmlApplicationContext:
    protected void loadBeanDefinitions(DefaultListableBeanFactory beanFactory) throws BeansException, IOException {
    	// Create a new XmlBeanDefinitionReader for the given BeanFactory.
    	XmlBeanDefinitionReader beanDefinitionReader = new XmlBeanDefinitionReader(beanFactory);

    	// Configure the bean definition reader with this context's
    	// resource loading environment.
    	beanDefinitionReader.setEnvironment(this.getEnvironment());
    	beanDefinitionReader.setResourceLoader(this);
    	beanDefinitionReader.setEntityResolver(new ResourceEntityResolver(this));

    	// Allow a subclass to provide custom initialization of the reader,
    	// then proceed with actually loading the bean definitions.
    	initBeanDefinitionReader(beanDefinitionReader);
    	loadBeanDefinitions(beanDefinitionReader);
    }
---
XmlBeanDefinitionReader:
    public int loadBeanDefinitions(Resource resource) throws BeanDefinitionStoreException {
        //加载xml资源
        return loadBeanDefinitions(new EncodedResource(resource));
    }

    public int loadBeanDefinitions(EncodedResource encodedResource) throws BeanDefinitionStoreException {

            InputStream inputStream = encodedResource.getResource().getInputStream();
            try {
                InputSource inputSource = new InputSource(inputStream);
                if (encodedResource.getEncoding() != null) {
                    //设置字符集
                    inputSource.setEncoding(encodedResource.getEncoding());
                }
                return doLoadBeanDefinitions(inputSource, encodedResource.getResource());
            }

    }

    protected int doLoadBeanDefinitions(InputSource inputSource, Resource resource)
            throws BeanDefinitionStoreException {
        try {
            int validationMode = getValidationModeForResource(resource);
            //根据xml文件，得到标准的Document对象
            //EntityResolver 主要用做验证用，可参考http://www.cnblogs.com/mjorcen/p/3642855.html
            Document doc = this.documentLoader.loadDocument(
                    inputSource, getEntityResolver(), this.errorHandler, validationMode, isNamespaceAware());
            return registerBeanDefinitions(doc, resource);

    }

    public int registerBeanDefinitions(Document doc, Resource resource) throws BeanDefinitionStoreException {
        //BeanDefinitionDocumentReader实际解析doc文档
        BeanDefinitionDocumentReader documentReader = createBeanDefinitionDocumentReader();
        int countBefore = getRegistry().getBeanDefinitionCount();
        documentReader.registerBeanDefinitions(doc, createReaderContext(resource));
        return getRegistry().getBeanDefinitionCount() - countBefore;
    }

```

**源码二：元素解析**
```Java8
DefaultBeanDefinitionDocumentReader:
    protected void doRegisterBeanDefinitions(Element root) {

		BeanDefinitionParserDelegate parent = this.delegate;
        //元素解析的代理类，主要的bean解析，以及一些自定义元素的解析
		this.delegate = createDelegate(this.readerContext, root, parent);
        ...
        //默认为空操作
		preProcessXml(root);
		parseBeanDefinitions(root, this.delegate);
		postProcessXml(root);

		this.delegate = parent;
	}

    protected void parseBeanDefinitions(Element root, BeanDefinitionParserDelegate delegate) {
         //默认命名空间为http://www.springframework.org/schema/beans，也就是xml文件中<bean>的标签
         if (delegate.isDefaultNamespace(root)) {
             NodeList nl = root.getChildNodes();
             for (int i = 0; i < nl.getLength(); i++) {
                 Node node = nl.item(i);
                 if (node instanceof Element) {
                     Element ele = (Element) node;
                     if (delegate.isDefaultNamespace(ele)) {
                         //处理默认<bean>标签
                         parseDefaultElement(ele, delegate);
                     }
                     else {
                         //处理自定义标签 (i.e.  AOP,CONTENT,JDBC)
                         delegate.parseCustomElement(ele);
                     }
                 }
             }
         }
         else {//处理自定义标签
             delegate.parseCustomElement(root);
         }
     }
```

**源码三：基本元素解析**
```Java8
DefaultBeanDefinitionDocumentReader // 默认标签处理
    private void parseDefaultElement(Element ele, BeanDefinitionParserDelegate delegate) {
        // import tag
        if (delegate.nodeNameEquals(ele, IMPORT_ELEMENT)) {
            importBeanDefinitionResource(ele);
        }
        // alias tag
        else if (delegate.nodeNameEquals(ele, ALIAS_ELEMENT)) {
            processAliasRegistration(ele);
        }
        // bean tag
        else if (delegate.nodeNameEquals(ele, BEAN_ELEMENT)) {
            processBeanDefinition(ele, delegate);
        }
        // beans tag （嵌套import／alias／bean）
        else if (delegate.nodeNameEquals(ele, NESTED_BEANS_ELEMENT)) {
            // recurse
            doRegisterBeanDefinitions(ele);
        }
    }

    protected void processBeanDefinition(Element ele, BeanDefinitionParserDelegate delegate) {
        //解析element，得到BeanDefinition，BeanDefinitionHolder中包含了BeanDefinition
        BeanDefinitionHolder bdHolder = delegate.parseBeanDefinitionElement(ele);
        if (bdHolder != null) {
            bdHolder = delegate.decorateBeanDefinitionIfRequired(ele, bdHolder);
            try {
                //注册BeanDefinition
                BeanDefinitionReaderUtils.registerBeanDefinition(bdHolder, getReaderContext().getRegistry());
            }
            catch (BeanDefinitionStoreException ex) {
                getReaderContext().error("Failed to register bean definition with name '" +
                        bdHolder.getBeanName() + "'", ele, ex);
            }
            // Send registration event.
            getReaderContext().fireComponentRegistered(new BeanComponentDefinition(bdHolder));
        }
    }
---
BeanDefinitionParserDelegate:
    public BeanDefinitionHolder parseBeanDefinitionElement(Element ele) {
        return parseBeanDefinitionElement(ele, null);
    }

    public BeanDefinitionHolder parseBeanDefinitionElement(Element ele, BeanDefinition containingBean) {
		String id = ele.getAttribute(ID_ATTRIBUTE);
		String nameAttr = ele.getAttribute(NAME_ATTRIBUTE);

		List<String> aliases = new ArrayList<String>();
		if (StringUtils.hasLength(nameAttr)) {
			String[] nameArr = StringUtils.tokenizeToStringArray(nameAttr, MULTI_VALUE_ATTRIBUTE_DELIMITERS);
			aliases.addAll(Arrays.asList(nameArr));
		}

		String beanName = id;
        .....

		if (containingBean == null) {
             //检查bean name的唯一性. 如果usedNames不存在时，则加入usedNames，否则抛出重名异常
			checkNameUniqueness(beanName, aliases, ele);
		}

        //解析Element,真正解析<bean>的地方
		AbstractBeanDefinition beanDefinition = parseBeanDefinitionElement(ele, beanName, containingBean);
		if (beanDefinition != null) {
            .....
			String[] aliasesArray = StringUtils.toStringArray(aliases);
			return new BeanDefinitionHolder(beanDefinition, beanName, aliasesArray);
		}

		return null;
	}

```

**源码四：处理扩展或自定义标签**

```Java8
BeanDefinitionParserDelegate：
    public BeanDefinition parseCustomElement(Element ele) {
        return parseCustomElement(ele, null);
    }

    public BeanDefinition parseCustomElement(Element ele, BeanDefinition containingBd) {
		String namespaceUri = getNamespaceURI(ele);
        //根据元素的命名空间得到NamespaceHandler
		NamespaceHandler handler = this.readerContext.getNamespaceHandlerResolver().resolve(namespaceUri);
		if (handler == null) {
			error("Unable to locate Spring NamespaceHandler for XML schema namespace [" + namespaceUri + "]", ele);
			return null;
		}
        //使用合适的NamespaceHandler解析元素
		return handler.parse(ele, new ParserContext(this.readerContext, this, containingBd));
	}

NamespaceHandlerSupport：
    public BeanDefinition parse(Element element, ParserContext parserContext) {
        return findParserForElement(element, parserContext).parse(element, parserContext);
    }
```

**源码四、一：处理扩展或自定义标签的处理类何处加载**

在上面{NamespaceHandler handler = this.readerContext.getNamespaceHandlerResolver().resolve(namespaceUri)}这句话加载

```Java8
BeanDefinitionParserDelegate：
    public BeanDefinition parseCustomElement(Element ele, BeanDefinition containingBd) {
        String namespaceUri = getNamespaceURI(ele);
        // 加载自定义标签处理类
        NamespaceHandler handler = this.readerContext.getNamespaceHandlerResolver().resolve(namespaceUri);
        ....
    }

DefaultNamespaceHandlerResolver：
    public NamespaceHandler resolve(String namespaceUri) {
         //得到Map<Element名称，NamespaceHandler处理类>
        Map<String, Object> handlerMappings = getHandlerMappings();
        //根据namespaceUri得到匹配的NamespaceHandler
        Object handlerOrClassName = handlerMappings.get(namespaceUri);
        if (handlerOrClassName == null) {
            return null;
        }
        else if (handlerOrClassName instanceof NamespaceHandler) {
            return (NamespaceHandler) handlerOrClassName;
        }
        else {
            String className = (String) handlerOrClassName;
            .....
        }
    }

    private Map<String, Object> getHandlerMappings() {
        if (this.handlerMappings == null) {
            synchronized (this) {
                if (this.handlerMappings == null) {
                    try {
                        //handlerMappingsLocation默认值：META-INF/spring.handlers
                        //spring就会从jar包中的Meta-INF/spring.handlers文件中得到处理各种不同命名空间元素的类
                        Properties mappings =
                                PropertiesLoaderUtils.loadAllProperties(this.handlerMappingsLocation, this.classLoader);
                        if (logger.isDebugEnabled()) {
                            logger.debug("Loaded NamespaceHandler mappings: " + mappings);
                        }
                        Map<String, Object> handlerMappings = new ConcurrentHashMap<String, Object>(mappings.size());
                        CollectionUtils.mergePropertiesIntoMap(mappings, handlerMappings);
                        this.handlerMappings = handlerMappings;
                    }
                    catch (IOException ex) {
                        throw new IllegalStateException(
                                "Unable to load NamespaceHandler mappings from location [" + this.handlerMappingsLocation + "]", ex);
                    }
                }
            }
        }
        return this.handlerMappings;
    }
```
Spring是如何解析自定义元素(i.e. <context:component-scan>,<aop:config> )。通过源码分析看出spring在解析的过程中，会去收集"spring. * .jar/META-INF"下的 spring.handers,spring.schemas文件，这2个文件就是指明了解析spring中自定义标签的Namespace类。如果自己开发Spring组件，需要增加新的标签，也可以按照这个机制。

### BeanDefinition 在IOC中的注册

```Java8
DefaultListableBeanFactory:
public void registerBeanDefinition(String beanName, BeanDefinition beanDefinition)
        throws BeanDefinitionStoreException {
    ....
    BeanDefinition oldBeanDefinition;

    oldBeanDefinition = this.beanDefinitionMap.get(beanName);
    if (oldBeanDefinition != null) {
        ....
        this.beanDefinitionMap.put(beanName, beanDefinition);
    }
    else {
        if (hasBeanCreationStarted()) {
            // Cannot modify startup-time collection elements anymore (for stable iteration)
            synchronized (this.beanDefinitionMap) {
                this.beanDefinitionMap.put(beanName, beanDefinition);
                List<String> updatedDefinitions = new ArrayList<String>(this.beanDefinitionNames.size() + 1);
                updatedDefinitions.addAll(this.beanDefinitionNames);
                updatedDefinitions.add(beanName);
                this.beanDefinitionNames = updatedDefinitions;
                if (this.manualSingletonNames.contains(beanName)) {
                    Set<String> updatedSingletons = new LinkedHashSet<String>(this.manualSingletonNames);
                    updatedSingletons.remove(beanName);
                    this.manualSingletonNames = updatedSingletons;
                }
            }
        }
        else {
            // Still in startup registration phase
            this.beanDefinitionMap.put(beanName, beanDefinition);
            this.beanDefinitionNames.add(beanName);
            this.manualSingletonNames.remove(beanName);
        }
        this.frozenBeanDefinitionNames = null;
    }

    if (oldBeanDefinition != null || containsSingleton(beanName)) {
        resetBeanDefinition(beanName);
    }
}
```

参考：
http://www.blogjava.net/heavensay/archive/2013/10/28/405699.html
http://kyfxbl.iteye.com/blog/1610255 (小读spring ioc源码)
http://blog.csdn.net/cutesource/article/details/5864562 (基于Spring可扩展Schema提供自定义配置支持)
