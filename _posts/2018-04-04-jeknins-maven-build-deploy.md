---
layout: post
title:  "Jenkins、Maven部署踩坑"
date:   2018-04-04 10:58:52
categories: maven jenkins deploy
published: true
comments: true
thread: 20180404101155555
---
Jenkins、Maven部署踩坑
---

# 目录
- 第一部分 踩坑
  - 背景
  - 表现
- 第二部分 埋坑
  - 分析
  - 埋坑一
  - 埋坑二

# 第一部分 踩坑
## 背景
合作方daemon系统半自动部署，原有方式这样的
  1. 环境：svn + maven3.0.3 + nexus， 涉及`svn版本机器`、`maven私服`、`目标部署机器`
  2. 通过maven在`目标部署机器`上完成构建，构建的jar包形式运行的：
    - maven构建时，通过maven-jar-plugin2.4把pom文件中所有依赖打入jar包文件中的`META-INF/MANIFEST`文件中classpath属性
    - 在`目标部署机器`运行部署包构建shell时，通过maven-dependency-plugin2.5.1读取`META-INF/MANIFEST`文件中classpath中的jar包完成依赖拉取下载到启动jar所在目录下`dependency/`目录下
    - 在`目标部署机器`运行启动shell时，通过内部封装工具类以及main-class属性完成系统启动。

上面这种方式有一个缺点，要在每台部署的机器上完成maven构建过程，本着DRY原则（Don’t repeat yourself，不要做重复的事情），有一个小优化
  1. 环境：gitlab + jenkins + maven + nexus，涉及`gitlab机器`、`jenkins机器`、`maven私服`、`部署协调机器`、`目标部署机器`
  2. 在`jenkins机器`完成系统项目构建
  3. 在`部署协调机器`运行部署包构建shell，完成部署包构建并推送到目标机器上。
  4. `目标部署机器`上执行启动shell，，通过内部封装工具类以及main-class属性完成系统启动。

## 表现
如果系统要部署10个机器集群，改进后减少9次构建，节省大量build时间、大量网络、电量消耗，节省下来time和money我们可以去点一杯java下午茶了。  

然而剧情并没有按照剧本发展，执行完优化流程后，目标机器启动报错：`java.lang.ClassNotFound：xxx`

# 第二部分 埋坑
## 分析
根据报错信息初步判定依赖包`a-SNAPSHOT.jar`没有加载，解压启动的`start.jar`发现`META-INF/MANIFEST`的classpath中jar信息和 `dependency/`目录下`a.20180404-SNAPSHOT.jar`不一致导致没有加载到。 解决方式两个：
  1. 构建jar包时，把时间戳加到`META-INF/MANIFEST`的classpath中
  2. `META-INF/MANIFEST`的classpath中依赖包不带时间戳，下载到 `dependency/`目录时把时间戳去掉。

## 埋坑一 ❌
1. 如果要加时间戳进jar包的`META-INF/MANIFEST`中，需要考虑项目打包配置，看maven-jar-plugin配置，发现有个属性useUniqueVersions=true时，会加入时间戳。
2. 在pom插件中实施后，jenkins构建后的jar包`META-INF/MANIFEST`文件中classpath内容中发现依赖其他项目的jar的时间戳是有的；但是同项目下的依赖jar是不带时间戳的。
  - maven构建后会把jar先装载到本地repository，然后上传到maven私服服务器。由于是快照版本，按照maven规范，上传私服时快照版本会带一个时间戳在上；但刚装载到本地的快照包不带时间戳
  - maven-jar-plugin构建jar对`META-INF/MANIFEST`写入classpath属性时，会遍历pom文件中依赖并根据对本地repository中的jar包名称写入。

从上面描述看这个方式走不通。不能保证jar打包时，manifest文件中时间戳必备。

## 埋坑二 ✅
1. pom中useUniqueVersions=false，验证jar打包后时间戳是不带的
2. 在`部署协调机器`通过maven-dependency-plugin2.5.1下载插件时发现下载的jar包是带有时间戳的，看源码发现高版本插件多一个属性useBaseVersion会把时间戳去掉，并且默认配置是去掉的。
3. 升级maven-dependency-plugin版本到2.8后，拉取jar包不带时间戳，成功解决。
4. 在`目标部署机器`上执行启动shell，启动成功。
