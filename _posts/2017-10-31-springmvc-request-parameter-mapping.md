---
layout: post
title:  "SpringMVC参数映射"
date:   2017-10-31 10:58:52
categories: request springmvc
published: true
comments: true
thread: 20171031101155555
---
SpringMVC参数映射
---

1. HandlerExecutionChain: 配置了handler的拦截器链，还有具体的controller处理方法
2. HandlerAdapter
  - ServletWebRequest 封装request
  - HandlerMethod 封装
    - setDataBinderFactory : 数据绑定
    - setHandlerMethodArgumentResolvers ：参数映射&messageConverter
    - setHandlerMethodReturnValueHandlers： messageConverter
    - setParameterNameDiscoverer
  - invocableMethod.invokeAndHandle
    - getMethodArgumentValues
      - argumentResolvers.resolveArgument ： 数据绑定、参数映射、消息转换
      - doInvoke 执行Controller方法
