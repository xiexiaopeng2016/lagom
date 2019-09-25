# 组件技术

作为一个完整的微服务平台，Lagom汇集了一系列技术并在这些技术之上增加了价值。Lagom使用和支持的某些库，工具和服务器是在[Lightbend](https://lightbend.com)开发的，其他是第三方和开源的。在使用Lagom进行开发时，您还可以利用以下技术:

* Play框架-Lagom在Lightbend的Web框架Play框架之上实现。这是一个实现细节，在开发简单的微服务时不会直接与您有关。但是，高级用户可以直接调用某些Play API。如果您有要向其添加微服务的现有Play Framework应用程序，则Lagom将为该用例提供支持。
* Play框架 --- Lagom在Lightbend的Web框架[Play框架](https://www.playframework.com)之上实现。这是一个实现细节，在开发简单的微服务时不会直接与您有关。但是，高级用户可以直接调用某些Play API。如果您有要向其添加微服务的现有Play Framework应用程序，则Lagom将为该用例提供支持。

* Akka --- Lagom [[持久化|PersistentEntity]], [[发布-订阅|PubSub]], and [[集群|Cluster]] 是在Lightbend的工具包[Akka](https://akka.io/)的基础上实现的, 该工具包用于构建并发，分布式和弹性消息驱动的应用程序。(这是一个实现细节，在开发简单的微服务时并不直接与您有关。但是，您也可以直接调用[[Akka APIs|Akka]].)

    * 为了跨多个服务器扩展您的微服务，Lagom通过[Akka Cluster](https://doc.akka.io/docs/akka/2.6/cluster-usage.html)提供集群。

    * 如[[实施服务|ServiceImplementation]]中所述，Lagom服务可以是"简单的"或"流式的"。流式, 异步Lagom服务建立在[Akka Streams](https://doc.akka.io/docs/akka/2.6/stream/index.html)之上。

* [Lightbend平台](https://www.lightbend.com/lightbend-platform) 订阅者可以使用额外的组件来操作和生产——增强他们的系统。

    * Akka [Split Brain Resolver](https://doc.akka.io/docs/akka-enhancements/current/split-brain-resolver.html) 处理网络故障和系统崩溃。

    * [Lightbend Telemetry](https://developer.lightbend.com/docs/telemetry/current/home.html) and [Lightbend Console](https://developer.lightbend.com/docs/console/current/) 可让您查看系统的运行状况, 可用性和性能。

    * 查看 [[Lightbend Platform|LightbendPlatform]]了解更多详细信息.

* Cassandra --- 默认情况下, 需要持久保存数据的Lagom微服务使用[Cassandra](https://cassandra.apache.org)实例, 其运行为开发环境的一部分。您也可以使用现有的[[Cassandra Server|CassandraServer]]数据库或其他类型的数据库。有关更多信息，请参见[[管理数据持久性|ES_CQRS]]。

* Guice --- 类似Play，Lagom使用[Guice](https://github.com/google/guice)进行依赖项注入。

* SLF4J & Logback --- Lagom使用[SLF4J](https://www.slf4j.org/)进行日志记录，并由[Logback](https://logback.qos.ch/)作为默认日志记录引擎进行支持。有关更多信息，请参见[[Logging]]。

* Typesafe Config 库 --- Lagom及其许多组件技术是使用[Typesafe Config](https://github.com/typesafehub/config)库进行配置的。配置文件格式为[HOCON](https://github.com/typesafehub/config/blob/master/HOCON.md)，它是JSON的强大而富有表现力的超集。

* 序列化 --- Lagom推荐的序列化格式为JSON。用于Java的JSON序列化和反序列化的默认引擎是[Jackson](https://github.com/FasterXML/jackson)和Scala的[Play JSON](https://www.playframework.com/documentation/2.6.x/ScalaJson)。还支持其他序列化格式。
