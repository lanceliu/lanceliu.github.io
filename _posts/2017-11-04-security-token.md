---
layout: post
title:  "token引发的安全思考"
date:   2017-11-04 10:58:52
categories: token security
published: true
comments: true
thread: 20171104101155555
---
登录TOKEN
---

http api认证机制是通过token的.
## 一、Token是什么
Token是服务端生成的一串字符串，当用户登录后，服务器生成一个Token返回给客户端，以后客户端只需带上Token前来请求数据即可，无需再次带上用户名和密码。

## 二、生成机制
在服务器段对一组有意义的字符串使用对称的方式加密。比如对`用户名`、或者`用户名+时间戳`、或者`登录历史+用户ID+时间戳`进行对称加密。

## 三、验证机制
接收到客户端传输过来的token，服务端通过加密密钥解密获取到明文字符串。

## 四、Token应用场景
还经常用于设计用户认证和授权系统，甚至实现Web应用的单点登录。
优点：
  1. 平台内可以跨系统验证
  2. token是自描述的。减少对db的依赖。
  3. 支持cookie时，可以通过 httpOnly cookie来完成认证。可以保证不被客户端修改
  4. 不支持cookie的情况，可以通过url／header区域来完成认证。又被客户端修改的可能
