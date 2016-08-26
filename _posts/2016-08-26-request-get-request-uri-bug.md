---
layout: post
title:  "request.getRequestURI return url with hash symbol"
date:   2016-08-10 09:17:52
categories: servlet http
published: true
comments: true
thread: 20160826131355555
---
有一个单页面应用，页面跳转通过修饰前端的url的hash来完成，比如`domain/#!index` 跳转到产品页 `domain/#!product`。
如果只是通过页面点击访问是没有问题，但是如果刷新当前带hash的url，则会报404错误。应用服务器和客户端环境
Server Environment：
```
web server: nginx1.6
app server: tomcat7.x
servlet: 3.0
dispatch: Spring MVC
```

Client Environment:
```
machine: iPhone
os:      9.3.4
browser: QQ browser any version
```

## 1. 问题排查
1. 使用iPhone其他os版本，使用QQ浏览器访问问题不存在，锁定为操作系统9.3.4和QQ浏览器问题
2. 关闭nginx后问题依然存在，排除web server影响。
3. 使用tomcat 8.x部署应用，问题依然存在。更换app server为jetty后问题消除。

问题锁定为：  iOS 9.3.4 + QQ browser + tomcat 7.x/8.x

在`iOS 9.3.4 + QQ browser `客户端访问`domain/#!index`时，debug环境下request.getRequestURI理论上应该返回`/`, 但是却发现返回了`/#!index`. spring MVC的路由中是没有注册`/#!index`这个路由的，所以报错`org.springframework.web.servlet.PageNotFound.noHandlerFound No mapping found for HTTP request with URI [/#!index] in DispatcherServlet with name 'servlet'`.

由于问题比较小众，单页面应用也只是在微信下的h5，所以不做修复。如何提交bug给qq浏览器和ios，如果有知道的可以帮忙提交一下这个issue。
