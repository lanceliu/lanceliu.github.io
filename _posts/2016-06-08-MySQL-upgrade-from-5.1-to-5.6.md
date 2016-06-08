---
layout: post
title:  "Mysql 5.1.73升级到5.6.14"
date:   2016-06-08 15:47:52
categories: mysql upgrade
published: true
comments: true
thread: 20160608154752555
---

只读事务在MySQL5.6中引入，改进了创建视图快照的开销，减少了持有trx_sys->mutex的时间，这有利于提升只读性能；这一点已经广为人知。

    鉴于我司版本极低，所以亟需升级，接下来把本次升级的步骤和填坑记录下来：

升级用的在线环境：
- Centos 6.5
- MySql5.1.73
- Seafile（为什么要提到他，在后面有提到他）

## 第一步：准备安装文件
```shell
wget http://dev.mysql.com/get/Downloads/MySQL-5.6/MySQL-shared-5.6.14-1.el6.x86_64.rpm/from/http://cdn.mysql.com/

wget http://dev.mysql.com/get/Downloads/MySQL-5.6/MySQL-client-5.6.14-1.el6.x86_64.rpm/from/http://cdn.mysql.com/

wget http://dev.mysql.com/get/Downloads/MySQL-5.6/MySQL-server-5.6.14-1.el6.x86_64.rpm/from/http://cdn.mysql.com/
```

## 第二步：备份SQL数据
由于我们的Mysql数据目录在在Centos环境的默认数据目录[/var/lib/mysql], 直接copy机器上的备份目录。

## 第三步：卸载旧版本
yum remove mysql mysql-server mysql-libs compat-mysql51
> 这时会删除掉python的一个连接mysql的依赖［MySQL-python］

## 第四步：安装新版本
```shell
rpm -ivh MySQL-shared-5.6.14-1.el6.x86_64.rpm
rpm -ivh MySQL-server-5.6.14-1.el6.x86_64.rpm
rpm -ivh MySQL-client-5.6.14-1.el6.x86_64.rpm

service mysql start
```
OK，到这里我们一般就认为是升级完毕了，开森。
少年太年轻了……

踩坑开始：
- 由于一个schema使用了自定义函数，调用时报错“Cannot load from mysql.proc”
    - ~~mysql_upgrade -uroot -p密码~~(**记住不需要执行该命令**)
    -  ALTER TABLE `proc` MODIFY COLUMN `comment`  text CHARACTER SET utf8 COLLATE utf8_bin NOT NULL AFTER `sql_mode`;
    - 该坑已填。
- 开启seafile服务，居然提示我创建新的管理员用户，连接客户端后从seafile相关的schema中任何一个表中查询时，都提示找不到该表。推测被第一个坑中不应该执行的命令给破坏掉了。
    - 还好我们在第二步中已经备份数据。
    - 解压备份数据替换［/var/lib/mysql］, 启动报错“MySQL ERROR! The server quit without updating PID file”，查看/etc/my.cnf中的错误日志输出路径（如果/etc/my.cnf没有时，错误日志路径/var/lib/mysql/[机器名].err）,查看日志
        - unknown variable 'default-character-set=utf8'
        - Operating system error number 13（文件操作权限), 由于解压后的文件归属和分组不是mysql， chown mysql:mysql -R /var/lib/mysql后成功。

至此所有坑已填。
