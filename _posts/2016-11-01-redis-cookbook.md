---
layout: post
title:  "Redis 入门"
date:   2016-11-01 14:49:52
categories: redis
published: true
comments: true
thread: 20161101145055555
---
Redis
---

## 一. 启动配置
安装完redis后。运行`redis-server`启动服务器。 执行该命令时可以设置端口号，是否开启持久化，日志级别等，为了便于启动也支持使用配置文件启动，如：
lin
```shell
$ redis-server /path/to/redis.conf
```
redis 是一个字典结构的存储服务器，一个redis实例默认支持16个数据库，对外都是以一个从0开始的递增数字命名。如果要更换数据库，

```shell
 redis> select 1
 OK
```

redis 非常轻量级，一个空redis实例占用内存只有1mb左右。

## 二. 入门
主要支持的几种数据类型：string，hash，list，set，zset

### 命令

#### 键名规范
（1）？ 匹配一个字符
（2）* 匹配任意个字符
（3）[] 匹配中括号内的任一字符，可以用-来表示一个范围
（4）\x 匹配字符x，用于转义符号

字符串类型是最基本的数据类型，最大容量为512M

#### 具体命令

KEYS * 列出所有键

EXISTS [key] 判断键是否存在

DEL [key……] 删除一个或多个键。 Del命令不支持通配符

type：获得键值的数据类型，格式是type key

rename：改名，格式是rename oldKey newKey

renamenx：如果不存在则改名，格式是rename oldKey newKey


incrby key increment 增加指定的整数

decr key increment 减少指定的整数

incrbyfloat key increment 增加制定浮点数

append key Value 向尾部追加值

strlen key 获取字符串长度

mget key [key...]
mset key value [key value ...] 同时获取／设定多个值

getbit key offset
setbit key offset value
bitcount key [start][end]  可以获得字符串类型键中值是1的二进制位个数。
bitop operation destkey key [key ...]

## 三、事务
redis的事务没有关系数据库事务提供的回滚功能。
- 防范语法错误。保证键名规范。
 key = 表名:主键名:列名
