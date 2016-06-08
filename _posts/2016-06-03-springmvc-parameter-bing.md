---
layout: post
title:  "SpringMVC Paramter Bind FOR PUT DELETE METHOD"
date:   2016-06-03 23:23:52
categories: spring springmvc put tomcat
published: true
comments: true
thread: 20160603232352555
---
从http1.1开始就有put，delete方法了。
今天遇到一个奇怪的问题，当使用detele或者put method使用form－data请求，但是获取不到请求参数。
问题出现的环境：

app－server：tomcat
web框架：springMVC
抓包工具：postman和fiddler


首先通过搜索引擎寻找方案，都说是浏览器不支持put／delete方法，需要通过一个filter来做请求转换。
客户端发起请求时，都适用post方法，但是传入一个额外的参数［_method]='put'||'delete', 在filter中完成具体的
springmvc路由分发。这个不是想要的解决方案，ignore。

然后看springMVC是否支持，测试证明支持PUT／DELETE方法，只是在参数绑定是没有葱request获取到value，但是葱request的inputstream中发现是有传入form data键值队的。
```java
byte[] bytes = IOUtils.toByteArray(httpServletRequest.getReader(), "UTF-8");
String formDataStr = new String(bytes);
```

排查APPServer配置。寻找tomcat配置，bing.com/baidu.com给出的答复都是在conf／web.xml 增加 init－param ［readonly］＝false来让
tomcat支持put／delete方法，试验无效。查询tomcat配置发现在conf／server.xml 中的connector选项中,增加解析requestbody中参数的行为
```xml
<Connector port="8080" protocol="HTTP/1.1"
       connectionTimeout="20000"
       parseBodyMethods="POST,PUT,DELETE"
       redirectPort="8443" URIEncoding="UTF-8" />
```
加入该配置后，tomcat重新启动，发现参数影射成功。OH yeah。


# SpringMVC 参数绑定机制
```java
org.springframework.web.method.support.InvocableHandlerMethod

    /**
	 * Get the method argument values for the current request.
	 */
	private Object[] getMethodArgumentValues(NativeWebRequest request, ModelAndViewContainer mavContainer,
			Object... providedArgs) throws Exception {

        // 在spring加载时解析出来的, 相当于spring的BeanDefinition
		MethodParameter[] parameters = getMethodParameters();
        // 获取参数对应的值
		Object[] args = new Object[parameters.length];
		for (int i = 0; i < parameters.length; i++) {
			MethodParameter parameter = parameters[i];
			parameter.initParameterNameDiscovery(this.parameterNameDiscoverer);
			GenericTypeResolver.resolveParameterType(parameter, getBean().getClass());
			args[i] = resolveProvidedArgument(parameter, providedArgs);
			if (args[i] != null) {
				continue;
			}
			if (this.argumentResolvers.supportsParameter(parameter)) {
				try {
                    // argumentResolvers注册了一些参数处理器，传入参数后会委托给具体的处理器来处理，以前处理的逻辑会被缓存
					args[i] = this.argumentResolvers.resolveArgument(
							parameter, mavContainer, request, this.dataBinderFactory);
					continue;
				}
				catch (Exception ex) {
					if (logger.isTraceEnabled()) {
						logger.trace(getArgumentResolutionErrorMessage("Error resolving argument", i), ex);
					}
					throw ex;
				}
			}
			if (args[i] == null) {
				String msg = getArgumentResolutionErrorMessage("No suitable resolver for argument", i);
				throw new IllegalStateException(msg);
			}
		}
		return args;
	}

```
