---
layout: post
title:  "Netty实战学习笔记一"
date:   2017-07-14 10:58:52
categories: netty
published: true
comments: true
thread: 20170714101155555
---
Netty实战学习笔记一
---
# 一、Netty主要构成
- Channel
    - 到有IO操作能力的实体的连接载体.传入／传出数据的载体，可以被打开／关闭，连接／断开。如读写操作.
- 回调
    - ChannelFutureListener / ChannelHandler
- Future
    - 异步操作的占位符，在未来某个时刻完成，并提供对其结果的访问。
- 事件和ChannelHandler

# 二、Netty DEMO
## 2.1 Netty服务器实现
至少要包含两部分
- 至少一个 ChannelHandler。实现服务器对客户端接口数据的处理，也就是业务逻辑。
- Bootstrap--配置服务器的启动代码。
    - 将服务器绑定到要监听连接请求的端口上。
    - 配置channel，将有关的入站消息通知给ChannelHandler

## 2.2 Netty客户端实现
步骤：
- 连接到服务器
- 发送消息
- 等待服务器反馈
- 关闭连接

# 三、Netty组件和设计
从两个角度来探讨Netty：类库和框架， 这两者对于使用Netty编写高效、可重用和可维护的代码来说，缺一不可。
Netty解决了两个相应的关注领域：技术和体系结构
- 基于NIO的异步和事件驱动，保证了高负载下的应用程序性能的最大化和伸缩性
- 包含了一组设计模式，将应用程序逻辑从网络层解耦，简化了开发，更容易测试。


## 3.1 Netty网络抽象的代表
- Channel--Socket
    - 基本的IO操作（bind(),connect(),read(),write()），依赖于底层网络传输所提供的原语。
    - Channel接口提供的AIP降低了直接使用Socket的复杂性
- EventLoop--控制流、多线程处理、并发
    - 用于处理连接的生命周期中所发生的事件。
    - Channel、EventLoop、Thread、EventLoopGroup之间的关系。
        - 一个Channel在他的生命周期只注册一个EventLoop。 N：1
        - 一个EventLoop可能被分配给一个或多个Channel。 1:N
        - 一个EventLoop在他的生命周期中只在它专有的Thread上被处理，且只和一个Thread绑定。 1:1
        - 一个EventLoopGroup包含一个或多个EventLoop。 1:N
- ChannelFuture--异步通知
    - 将来要执行的操作结果的占位符

## 3.2 管理数据流以及执行应用程序处理逻辑的组件
- ChannelHandler
    - Netty主要组件，充当了所有出入站数据的应用程序逻辑的容器
    - 方法是由网络事件触发的
    - 可专门用于几乎任何类型的动作，例如：数据格式转换、或处理转换过程中的异常
- ChannelPipeline
    - ChannelHandler Chain的容器
    - Channel被创建时，会自动分配到专属的ChannelPipeline

ChannelHandler安装到ChannelPipeline的过程如下：
- ChannelInitializer实现注册到ServerBootstrap中
- ChannelInitializer.initChannel()方法被调用是，ChannelInitializer将在ChannelPipeline中安装一组自定义的ChannelHandler
    - 分配一个ChannelHandlerContext， 可以用于获取Channel，主要用于写出站数据
- ChannelInitializer将自己从ChannelPipeline中移除

Netty中有两种发送消息的方式
- 直接写到Channel中：  导致消息从Pipeline的尾部开始流动
- 也可以写到和ChannelHandler关联的ChannelHandlerContext：Con 高Pipeline的下一个ChannelHandler开始流程


# 3.2.1 ChannelHandler && Adapter
Netty以Adapter形式提供了大量的ChannelHandler实现， 简化了应用程序处理逻辑的开发过程。常见的Adapter类：
- ChannelHandlerAdapter
- ChannelInboundHandlerAdapter
- ChannelOutboundHandlerAdapter
- ChannelDuplexHandler

用途：
- 将数据从一种格式转换为另一种格式
- 提供异常的通知
- 提供Channel变为活动的或者非活动的通知
- 提供当Channel注册到EventLoop或者EventLoop注销时的通知
- 提供有关用户自定义事件的通知

常见的ChannelHandler子类型有三类：编码器、解码器和SimpleChannelInboundHandler<T>(ChannelInboundHandlerAdapter的子类)

# 3.3 Bootstrap
Bootsrap类为application的网络层提供了容器，涉及将一个进程绑定到某个指定的端口，或者将一个进程连接到另一个运行运行在某个指定主机的指定端口上的进程。
有两种类型Bootstrap类
- Bootstrap
    - 想要连接到远程节点的客户端应用程序所使用的
    - EventLoopGroup的数目=1
- ServerBootstrap
    - 绑定到一个端口，因为服务器必须要监听连接
    - EventLoopGroup的数目可以=1，也可以=2. 需要两个Group的原因？因为服务器需要两组不同的channel
        - 第一组只包含一个ServerChannel，代表服务器自身的已经绑定到本地端口监听套接字。该Group将分配一个EventLoop为传入的连接请求创建Channel
        - 第二组将包含所有已创建用来处理传入客户端连接的Channel。

# 4 传输
传输方式：
- OIO- 阻塞传输
- NIO- 非阻塞传输
- Local - JVM内部的异步通讯
- Embeded - 测试你的ChannelHanlder

NIO四种状态 以及可获得通知
- OP_ACCEPT 新的Channel已被接受并且就绪
- OP_CONNECT 连接已经完成
- OP_READ 有已经就需的可供读取的数据
- OP_WRITE 可用于写数据

选择器（Selector）运行在一个线程上（检查状态变化&对变化做出响应响应），在应用程序对状态的改变作出响应之后，选择器会被重置，并重复这一过程。
