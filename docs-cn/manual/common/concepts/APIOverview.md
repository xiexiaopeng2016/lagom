# API 概述

Lagom同时提供Java和Scala API。Java API以Java 8为目标，并假定熟悉这些功能, 比如[lambdas](https://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html)，[default methods](https://docs.oracle.com/javase/tutorial/java/IandI/defaultmethods.html)和[`Optional`](https://docs.oracle.com/javase/8/docs/api/java/util/Optional.html)。Lagom的大多数是在Scala中实现的。但是，即使您使用Java API进行开发，这也是您不应该关心的实现细节。

Lagom的[[表达性服务接口声明|ServiceDescriptors]]使开发人员可以快速定义接口并立即开始实现它们。

使用Lagom进行开发时，将使用的重要API包括：

* [[Service API|ServiceDescriptors]] --- 提供一种声明和实现供客户端使用的服务接口的方式。为了位置透明，客户端通过服务定位器发现服务。Service API除了同步请求-响应调用之外，还支持服务之间的异步流。
* [[Message Broker API|MessageBrokerApi]] --- 提供分布式发布-订阅模型，其服务可用于通过主题共享数据。主题只是允许服务推入和拉出数据的渠道。
* [[持久性API|PersistentEntity]] --- 为存储数据的服务提供源于事件的持久性实体。还提供了命令查询责任隔离(CQRS)读取端支持。Lagom管理持久性实体在节点群集中的分布，从而实现分片和水平缩放。Cassandra是作为现成的数据库提供的，但是支持其他DB。
