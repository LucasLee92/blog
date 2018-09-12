---
title: learn_note_netty_in_action
tags:
    - 学习笔记
    - netty
---

## netty概念认知
### Channel, EventLoop, and ChannelFuture

* Channel: Sockets
* EventLoop: 控制流，多线程，并发
* ChannelFuture: 异步通知

#### interface Channel
基本`I/O`操作依赖于底层网络协议，在java代码是`Socket`。而Netty中的`Channel`则是简化了`Socket`的复杂性，同时也是扩展Channel的底层依赖。

#### interface EventLoop
`EventLoop`定义了Netty在连接过程中对事件处理的核心抽象

* `Channels`, `EventLoops`, `Threads`, and `EventLoopGroups`的高阶抽象
    * 一个`EventLoopGroup`包含一个或多个`EventLoops`
    * 一个`EventLoop`的生命周期都绑定在一个线程上
    * 一个`EventLoop`执行的所有`I/O`事件，都由它指定的线程执行
    * 一个`Channel`注册所有生命周期在一个单独的`EventLoop`
    * 一个`EventLoop`可能分配给一个或多个`Channels`
    * 在一个给定`Channel`上的`I/O`操作都是由同一个线程操作，事实上消除了异步的需要

#### interface ChannelFuture
所有的`I/O`操作在Netty上都是异步的。Netty对异步结果处理采用的方式是提供`ChannelFuture`, 它的`addListener`方法注册了一个`ChannelFutureListener`当一个操作完成的时候去被通知调用


### ChannelHandler and ChannelPipeLine
#### interface ChannleHandler
从开发者的角度而言，Netty的主要部件是`ChannelHandler`，担当了所有对出入数据的应用逻辑的容器。这可能是因为`ChannelHandler`方法和网络事件绑定。事实上一个`ChannelHandler`可以指定给任何动作。例如数据转换或者是异常捕捉

#### Interface ChannelPipeline
一个`ChannelPipeline`提供了一个容器容纳一系列的`ChannelHandlers`和定义了 API 用于在处理链上传递出入数据。当一个`Channel`创建的时候，会被赋值一个自己的`ChannelPipeline`
下面是赋值过程：
* 一个`ChannelInitializer`实现注册在一个`ServerBootstrap`
* self:`serverBootstrap`使用`ChannelInitializer`初始化`Channel`，`Channel`同时会带有`ChannlePipeline`，而`ChannleHandler`则会按顺序放在`ChannelPipeline`
* 当`ChannelInitializer.initChannel()`被调用，`ChannelInitializer`安装一系列自定义的`ChannelHandlers`在`pipeline`
* `ChannelInitializer`会从`ChannelPipeline`移除自己
* self:多个`ChannelHandler`调用是当`ctx`触发某个动作后触发第一个`ChannleHandler`的对应方法，如何需要继续下去的话，该方法内部需要实现`ctx.fireXxx`，`ChannelHandlerContext`作为`Channel`的容器和类似信息中心的存在

#### ChannelHandler and ChannelHandlerAdapter
Netty存在着大量的`ChannelHandlers`，而且大部分的功能由其父类确定。Netty提供了一些默认的实现`Adapter Class`，用来简化app的执行逻辑。每个`ChannelHandler`都在把事件传递给下一个handler，而这些`Adapter Class`已经实现了这部分功能，开发要做的是聚焦在你需要关注的事件和方法

#### Encoders and decoders
当Netty接受一条信息是，



------------
























