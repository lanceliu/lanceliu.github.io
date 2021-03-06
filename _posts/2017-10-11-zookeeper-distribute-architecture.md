---
layout: post
title:  "Zookeepe-一分布式架构"
date:   2017-10-11 14:50:52
categories: zookeeper
published: true
comments: true
thread: 20171011145055555
---

Zookeeper 一 分布式架构
---
如果逻辑控制流在时间上重叠，那么她们就是并发的。分布式环境引入数据复制后，在对一个副本数据进行更新的同时，确保也能更新其他的副本，否则不同副本间的数据将不再一致。
### 1. 集中式 到 分布式
集中式系统
- 定义：每个终端或客户端机器仅仅负责数据的录入和输出，数据的存储与控制处理完全交由主机来完成
- 特点：部署结构简单

分布式系统
- 定义：一个硬件或软件组件分布在不同的网络计算机上，彼此之间仅仅通过消息传递进行通信和协调的系统。
- 特点
    - 分布性
    - 对等性--没有主从之分，所有计算机节点都是对等的
    - 并发性
    - 缺乏全局时钟
    - 故障总会发生--任何在设计阶段考虑到的异常情况，一定会在系统实际运行中发生。
- 分布式环境的各种问题
    - 通信异常
    - 网络分区 -- 网络异常导致系统中部分节点之间的网络掩饰不断增大，最后导致止部分节点之间能够进行正常通讯，而另一些节点则不能，这个现象叫做网络分区、就是俗称的“脑裂”。
    - 三态 -- 成功、失败与超时
        - 发送过程中超时
        - 响应反馈给发送方时超时发生消息丢失
    - 节点故障 -- 节点故障，比如宕机或僵死

### 2. ACID 到 CAP／BASE
ACID -- 四种事务隔离级别

CAP，分布式系统不可能同时满足一致性、可用性和分区容错性。
一致性、
可用性（有限时间内返回）
分布容错性

BASE

Basically Available
    - 响应时间上，稍增加
    - 功能上的损失，比如部分功能降级
Soft state
    - 与硬状态对应，允许系统中的数据存在中间状态，该中间状态存在也不会影响系统的整体可用性。
Eventually consistent
    - 因果一致性
    - 读己之所写
    - 会话一致性
    - 单调读一致性
    - 单调写一致性
