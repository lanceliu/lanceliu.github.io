---
layout: post
title:  "Zookeeper"
date:   2017-04-02 14:50:52
categories: zookeeper
published: true
comments: true
thread: 20170402145055555
---

Zookeeper
---


## 1. 安装

### zookeeper安装 && 启动

```shell
[ ~ $] wget http://mirrors.cnnic.cn/apache/zookeeper/zookeeper-3.4.9/zookeeper-3.4.9.tar.gz
[ ~ $] tar zxvf zookeeper-3.4.9.tar.gz
[ ~ $] cd zookeeper-3.4.9
[ ~ $] cp -rf conf/zoo_sample.cfg conf/zoo.cfg
[ ~ $] cd conf
[ ~ $] vim zoo.cfg   #修改zook.cfg里头的dataDir
[ ~ $] cd bin
[ ~ $] sh zkServer.sh start
```

# 2. 设计目标
简单：允许分布式进程协调一个共享的分层命名空间类似于标准文件系统。命名空间由数据寄存器组成-Znode。和设计为存储的文件系统不通，Zookeeper数据保持在内存中，意味着由高吞吐量和低延迟。
ZooKeeper的实现对高性能，高可用性，严格有序的访问非常重要。
高性能意味着可以用于大型分布式系统；高可用性避免单点故障；严格的顺序性意味着在客户端可以实现复杂的同步原语

可复制：

有序：Zookeeper用数字标记每一个更新，用它来反射出所有的事务顺序。随后的操作可以使用这个顺序去实现更高级的抽象，例如同步原件。


快：读比写多时，运行最好，读比比例大概10:1.

# 3. 编程指导
本指南的前四部分介绍了各种ZooKeeper概念的更高层次的讨论。为了了解ZooKeeper如何工作以及如何使用它们，这些都是必需的。它不包含源代码，但它确实会熟悉与分布式计算相关的问题。第一组中的部分是：
- 数据模型
- 会话
- 监听
- 一致性保障

接下来的四节提供实用的编程信息。这些是：
- 构建块：ZooKeeper操作指南
- 绑定
- 程序结构：带简单示例
- Gotchas：常见问题和故障排除


### 3.1 数据模型
命名规范 **略**

#### Znodes
Zookeeper树的每个节点叫做Znode。包含一个状态结构：
    - 数据变更的版本号
    - Acl变更
    - 时间戳

版本号和时间戳用于验证缓存并协调更新。

    > 在分布式应用程序工程中，单词节点可以指通用主机，服务器，组合成员，客户端进程等。在ZooKeeper文档中，znodes是指数据节点。服务器是指构成ZooKeeper服务的机器;法定对等体是指构成集合的服务器;客户端是指使用ZooKeeper服务的任何主机或进程。

znode是程序员需要注意的主要抽象。 Znode有几个特点，值得一提。
    - 监听
        - 客户端可以在znode上设置监听。对znode的更改会触发监听，然后清除监听。当监听触发时，ZooKeeper会向客户端发送通知。有关监听的更多信息，请参见“ZooKeeper监听”一节。
    - 数据获取
        - 存储在命名空间中的每个znode处的数据以原子方式读取和写入。读取获取与znode相关联的所有数据字节，写入替换所有数据。每个节点都有一个访问控制列表（ACL），它限制谁能做什么。Zookeeper不是设计成数据库或大对象存储。
        - 各种形式的协调数据的共同属性是它们相对较小：以千字节为单位。 ZooKeeper客户端和服务器实现具有完整性检查，以确保znode具有少于1M的数据，但数据应远远小于平均值。
        - 如果需要大量数据存储，处理此类数据的通常模式是将其存储在大容量存储系统（如NFS或HDFS）上，并将指针存储到ZooKeeper中的存储位置。
    - 临时节点
        - 只要创建znode的会话处于活动状态，就会存在这些znodes。当会话结束时，znode被删除。
        - 临时znodes不允许有孩子
    - 序列节点
        - 创建znode时，您也可以请求ZooKeeper在路径的末端附加递增的计数器。
        - 计数器对所在父节点是唯一的。
        - 计数器格式是 十位数字。不满10位时，左边用0补齐。 i.e. "<path>0000000001"。主要用来简化排序
        - 用于存储下一个序列号的计数器是由父节点维护的有符号int（4字节），当递增超过2147483647（导致名称“<path> -2147483647”）时，计数器将溢出。

#### Zookeeper 时间
Zookeeper有多种方式记录时间：
    - Zxid （Zookeeper事务ID）
        - 对ZooKeeper状态的每次更改都会以zxid（ZooKeeper Transaction Id）的形式标记盖戳。这显示了对ZooKeeper的所有更改的总排序。每个更改都将有一个唯一的zxid，如果zxid1小于zxid2，则zxid1发生在zxid2之前。
    - 版本号（对节点的每次更改将导致该节点的其中一个版本号的增加）。有三种版本号
        - version（znode的数据更改数）
        - cversion（znode的子项更改次数）
        - aversion（znode的ACL更改次数）
    - Ticks(刻度，当使用多服务器ZooKeeper时，定义事件的时间单位)
        - 例如状态上传，会话超时，对等体之间的连接超时等。
        - Tick时间仅通过最小会话超时（2次刻度时间）间接暴露;
        - 如果客户端请求超过 < 最小会话超时，服务器将告诉客户端会话超时实际上是最小会话超时。
    - Real time
        - 除了在znode创建和znode修改之后将时间戳放入统计结构中，ZooKeeper不使用实时或时钟时间。

#### ZooKeeper Stat Structure
每个zookeeper节点由以下字段组成，获取节点信息时，也会显示：
    - czxid 节点创建时 zxid
    - mzxid 节点最后一次修改时的 zxid
    - pzxid 节点子项的最后修改时的 zxid
    - ctime 节点创建时 的时间（毫秒）
    - mtime 节点最后修改时 时间
    - version 节点数据变动数
    - cversion 节点子项的变动数
    - aversion 节点ACL的变动数
    - ephemeralOwner 临时节点的会话id；非临时节点=0；
    - dataLength 节点数据项的长度
    - numChildren 节点的子项数

### 3.2 Zookeeper 会话


> 写了半天发现有个站点已经翻译过了， [zookeeper中文网](http://zookeeper.majunwei.com/document/3.4.6/DeveloperProgrammerGuide.html)
