<!--- Copyright (C) 2016-2019 Lightbend Inc. <https://www.lightbend.com> -->
# Lagom设计哲学

考虑一下JonasBonér(引用自[*Reactive Microservices Architecture: Design Principles for Distributed Systems*] (https://info.lightbend.com/COLL-20XX-Reactive-Microservices-Architecture-RES-LP.html))所标识的响应式微服务的一些基本要求：

以下Lagom特性促进了这些最佳实践：

Lagom默认情况下是异步的-它的API通过流式传输使服务间通信成为一流的概念。所有Lagom API均使用Akka Stream的异步IO功能进行异步流传输；Java API使用JDK8CompletionStage进行异步计算；Scala API使用Futures。

与传统的集中式数据库相比，Lagom支持分布式持久模式。我们鼓励（但不要求）基于事件的体系结构来实现数据持久性。实体持久化的默认模式利用事件源（ES）和命令查询责任隔离（CQRS）。管理数据持久性从高层次上解释了什么是事件源及其为何有价值。持久实体介绍了Lagom对事件源的实现。

Lagom提供了用于开发目的的服务注册表和服务网关的实现，以及用于管理客户端和服务器端服务发现的内部管道。注册和发现服务引入了这些概念。


* "*隔离* 是弹性和灵活性的前提，并且需要服务边界之间的异步通信 ..."
* "*自治服务*只能通过发布其协议/API来*承诺*其行为" 和 "要使服务变得位置透明，它必须是可寻址的。"
* "需要的是每个微服务对自己的状态及其持久性承担全部责任。"

以下Lagom特性促进了这些最佳实践:

* Lagom默认情况下是异步的 --- 它的API通过流式传输使服务间通信成为第一等的概念。所有Lagom API均使用[Akka Stream](https://akka.io/)的异步IO功能进行异步流传输; Java API使用[JDK8 `CompletionStage`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletionStage.html)进行异步计算；Scala API使用[Futures](https://www.scala-lang.org/files/archive/api/2.12.x/scala/concurrent/Future.html)。

* 与传统的集中式数据库相比，Lagom支持分布式持久模式。我们鼓励 --- 但不要求 --- 基于事件的体系结构来实现数据持久性。实体持久化的默认模式利用事件源(ES)和命令查询责任隔离(CQRS)。[[管理数据持久性|ES_CQRS]]从高层次上解释了什么是事件源及其为何有价值。 [[持久实体|PersistentEntity]]介绍了Lagom对事件源的实现。

* Lagom提供了用于开发目的的服务注册和服务网关的实现，以及用于管理客户端和服务器端服务发现的内部管道 [[注册和发现服务|ServiceDiscovery]]引入了这些概念。
