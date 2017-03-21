---
layout: post
title:  "mongodb 学习笔记"
date:   2017-02-22 14:50:52
categories: mongodb
published: true
comments: true
thread: 20170222145055555
---

# mongodb 学习笔记

### 终端命令
- 查看所有： show dbs
- 切换当前库： use [dbname]
- 创建库： db.[newDbName].insert({"name":"mongostudy"})
    - newDbName规范：全部小写；最多64字节；不能有空格
    - admin／local／config 三个库保留
- 删除库,需要先切换过去： db.dropDatabase()
- 删除集合（table）： db.[collectionName].drop()
    - collectionName rule: 不能‘system.’开头
