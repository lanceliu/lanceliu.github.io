---
layout: post
title:  "禅道can not find driver"
date:   2016-06-12 09:47:52
categories: mysql upgrade
published: true
comments: true
thread: 20160612094752555
---

端午假期回来后，禅道系统登录失败，报错“can not find driver by function connectByPDO.”
出现问题的环境：
1. Centos 6.5
2. Mysql5.6
3. PHP5.3.3
4. Apache

搜索引擎之后发现时连接数据库失败，php连接数据库需要pdo_mysql这个模块。

- 在"/etc/php.ini"中查看是否配置了pdo_mysql这个扩展.[extension=/usr/lib64/.../pdo_mysql.so], 进入目录后没有发现，可能是被误删除掉了。
- 重新安装pdo_mysql.so需要从网络下载重新编译打包安装，由于一些配置问题失败，该路径走不通。
- 由于是php对mysql的依赖，考虑安装php-mysql这个扩展， yum install php-mysql, 在上面配置的路径下发现pdo_mysql.so。
- 重新请求后发现还是报错，应该是系统没有重新加载这个扩展，重新启动apache［service httpd restart］,再次访问，成功进入。
