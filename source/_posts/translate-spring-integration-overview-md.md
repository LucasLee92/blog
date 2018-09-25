---
title: translate-spring-integration-overview
date: 2018-09-12 17:56:00
tags:
    - 学习笔记
    - 翻译
    - Spring框架
    - Spring Integration框架
---

[原文地址](https://docs.spring.io/spring-integration/reference/html/overview.html)

# iii. Overview Of Spring Integration Framework

## 3. Spring Integration Overview

### 3.3. Main Components
从垂直的角度来看，一个分层架构使关注点的分离变得简单，基于结构的层之间的调用解耦和。基于Spring的程序就是上述观点的典型案例，Spring框架和其组件为企业实践上述观点提供了强有力的基础。尽管有些相同点，基于消息的结构添加了一个水平的关注点。就像分层结构是一个非常典型和抽象的模型，消息系统也是遵循抽象的 pipes-and-filters 模型。 filters 代表任何能够生成或消费消息的组件， pipes 负责在 filters 之间传输消息用于在组件之间解耦和。需要重要说明一下的是，两个高度抽象的模型并不会相互排斥。用于 pipes 的底层消息组件仍然应该被封装成用结构来调用。同样的， filters 自身也应该用一个在应用service的上层来管理，类似于 web 层的位置

#### 3.3.1. Message
在 Spring Integration 中，Message 是一个包装类包装任意 Java Object 同时带有用于框架的元信息。由 payload 和 header 组成。payload 可以是任何类型，headers 带着必需的信息如 id, timestamp, correlation id, return address。 headers 也用于传递值与连接的协议。例如，创建一个 Message 从一个接收到的文件，文件名应该存储在 header 用于被上传组件访问。 同样的，如果一个 Message 的内容最终打算发送到一个 outbound Mail adapter，不同的属性应该被配置成一个上传流组件的 Message header。开发者也同样可以存储任意的键值对信息在 header。

#### 3.3.2. Message Channel（消息通道）
一个消息通道代表着 pipes-and-filters 模型中的 pipe 。生产者发送消息到通道，消费者从通道接收消息。消息通道因此解耦了消息组件，也提供了一个方便的用于拦截和监听消息的点。
一个消息通道可能是端到端的，也可以是发布/订阅模式的。端到端的通道，最多只有一个消费者可以接受到通道中的信息。发布/订阅通道，会尝试把消息广播到这个通道的所有订阅者。
端到端和发布/订阅定义了多少消费者最终会接收到消息，但是仍然有一个重要的考虑：通道应该缓存消息吗？在 Spring Integration 中，可轮训的通道能够使用队列缓存消息。缓存能够让队列能够限流被消费的消息量防止消费者过载。然而，这也添加了一些复杂程度，因为一个消费者将只能从一个可轮训的通道接收消息。另外，连接到订阅通道的消费就是简单的消息驱动。

#### 3.3.3. Message Endpoint（消息终端）
Spring Integration 的主要目标之一就是通过 IOC 简化企业集成解决方案的开发。这意味着开发者不应该直接去实现消费者和生产者，甚至不用去构建消息和调用消息通道的发送或接受操作。而是应该专注于用PO实现业务模型。然后通过说明性的配置，你能够把你的领域代码连接到消息框架中。代表这些链接的组建是消息终端。这并不意味着你必须直接连接你已经存在的应用代码。任何真实的企业集成解决方案都会需要一些代码关注与集成关注点，例如路由和转换。重要的是去获得在集成逻辑和业务逻辑之间的关注点分离。换句话说，就像 MVC 模型对于web程序，模型应该能提供一个简单而专注的层来转换入站请求到 service 层调用，然后转换 service 层返回结果到出站层回复。

