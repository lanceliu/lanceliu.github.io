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

# 5 ByteBuffer
Java NIO的ByteBuffer作为字节容器，与Channel进行交互。
Netty 使用ByteBuf替代ByteBuffer，一个强大的实现既解决了JDK API的局限性，又为网络应用开发者提供了更好的API。

## 5.1 ByteBuf API
Netty数据处理API通过两个组件暴露-ByteBuf和ByteBufHolder，优点：
- 可以被用户自定义的缓冲区类型扩展
- 通过内置的复合缓冲区类型实现了透明的零拷贝。
- 容量可以按需增长（类似StringBuilder)
- 在读和写这两种模式之间切换不需要调用ByteBuffer的flip（）方法
- 读和写使用了不同的索引
- 支持方法的链式调用
- 支持引用计数
- 支持池化

## 5.2 ByteBuf - 数据容器
维护了两个不同索引：一个readderIndex／writerIndex
- 空ByteBuf， readerIndex=writerIndex=0；
- readerIndex=writerIndex时，将会到达可以读取的数据的末尾
- 名称以read／write开头的ByteBuf方法会推进对应的索引
- 名称以set／get开头的操作则不会。

使用方式
- 堆缓冲区
    - 将数据存储在JVM堆空间，也被称为 backing array
    - 能在没有池化下提供快速的分配和释放
- 直接缓冲区
    - NIO在JDK1.4引入的ByteBuffer类允许JVM实现通过本地调用来分配内存
    - 之上的内容驻留在挥别垃圾回收的堆之外
    - 如果数据是在堆上分配的缓冲区，通过socket发送之前，JVM会在内部把缓冲区copy到直接缓冲区
    - 相对于基于堆的缓冲区，他们的分配和释放都较为昂贵
- 复合缓冲区

# 6 ChannelHanlder & ChannelPipeline
## 6.1.1 ChannelHandler
Channel的生命周期状态，状态改变时会生成对应事件，将会被妆发给ChannelPipline中的ChannelHandler
- ChannelUnregistered： 已被创建，但还未注册到EventLoop
- ChannelRegistered：已经注册到EventLoop
- ChannelActive：处于活动状态，已经连接到他的远程节点。可以接收和发送数据
- ChannelInactive：没有连接到远程节点

## 6.1.2 ChannelHandler的生命周期
ChannelHandler被添加到ChannelPipeline／从ChannelPipeline移除时会调用这些操作。这些方法中都接受一个ChannelHandlerContext
- handlerAdded
- handlerRemoved
- exceptCaught

如果一个消息被消费者或者丢弃了，并且没有传递给CHannelPipline中的下一个ChannelOutboundHandler, 就应该调用ReferenceCountUtil.release()

## 6.2 ChannelPipeline
ChannelHandlerContext使ChannelHandler和他的ChannelPipeline以及其他的ChannelHandler交互
- 通知其所属的ChannelPipeline中的下一个ChannelHandler
- 还可以动态的修改所属的ChannelPipeline：所包含的ChannelHandler的编排

# 7 EventLoop 和线程模型
在netty4中，所有的I/O操作和事件都由已经被分配给了EventLoop的Thread来处理。

# 8 Bootstrap
Bootstrap 把Netty的核心组件以及组件拼装起来。

Bootstrap / ServerBootstrap 都实现了 Cloneable接口？
- 有时候需要创建多个具有类似配置／完全相通配置的Channel
- 不想为每个Channel都创建一个新的引导类实例

# 9 Unit Test
