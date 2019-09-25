# 介绍Lagom概念

Kubernetes是一个开源系统，用于自动化容器化应用程序的部署，扩展和管理。
DC/OS是一种由Apache Messphere，Inc.创建和维护的基于Apache Mesos分布式系统内核的开源分布式操作系统。
支持主机之间的私有网络的任何云或本地部署环境。

Lagom框架包括支持您从开发到部署的库和开发环境：

* 在开发期间，一个命令将构建您的项目，并启动所有服务和支持的Lagom基础架构。修改代码时，它会热重载。开发环境使您可以在短短几分钟内提出一项新服务或加入现有的Lagom开发团队。
* 您可以使用Java或Scala创建微服务。Lagom为微服务之间的通信提供了特别无缝的体验。服务位置，通信协议和其他问题由Lagom透明处理，从而最大程度地提高了便利性和生产率。Lagom支持事件源和CQRS（命令查询责任隔离）来实现持久性。

* Lagom为您创建了服务客户端（基于服务描述符），这使得调用和测试端点变得极其容易。
* 在您选择的平台上进行部署，例如:
    * [Kubernetes](https://kubernetes.io), 是一个开源系统，用于自动化容器化应用程序的部署，扩展和管理。
    * [DC/OS](https://dcos.io/), DC/OS是一种由Apache Messphere，Inc.创建和维护的基于Apache Mesos分布式系统内核的开源分布式操作系统。
    * 支持主机之间的私有网络的任何云或本地部署环境。

设计具有高可伸缩性并在遇到意外故障时表现出弹性的微服务系统非常困难。没有Lagom之类的框架，您将需要处理高度分布式系统中固有的所有复杂线程和并发问题。通过按照设计使用Lagom，您可以避免许多这样的陷阱，同时提高生产力。但是，与其把所有的东西都扔掉，重新开始，Lagom允许您在现有约束条件内采用
[反应性架构](https://info.lightbend.com/COLL-20XX-Reactive-Microservices-Architecture-RES-LP.html)。例如，您可以创建以下微服务：

* 与遗留系统进行交互和/或替换单体应用程序功能。
* 将Cassandra用于持久化或您选择的数据库和/或与其他数据存储集成。(Lagom的持久化API默认情况下支持Cassandra，因为它提供了诸如分片和读取端支持之类的功能，这些功能在微服务系统中运行良好)

本节中的其余主题进一步介绍了:

* Lagom系统架构:
    * [[Lagom设计哲学|LagomDesignPhilosophy]]
    * [[多种语言系统|PolyglotSystems]]
* Lagom开发环境:
    * [[概述|DevelopmentEnvironmentOverview]]
    * [[构建哲学|BuildConcepts]]
    * [[组件技术|ComponentTechnologies]]
    * [[API概述|APIOverview]]
* 在系统中使用的模式:
    * [[设计您的微服务系统|MicroserviceSystemDesign]]
    * [[调整单个微服务的大小|MicroserviceDesign]]
    * [[内部和外部通信|InternalAndExternalCommunication]]
    * [[注册和发现服务|ServiceDiscovery]]
    * [[使用不可变的对象|Immutable]]
    * [[管理数据持久性|ES_CQRS]]
    * [[事件源的优势|ESAdvantage]]
    * [[将读取与写入分开|ReadVsWrite]]
    * [[部署弹性，可扩展的系统|ScalableDeployment]]


@toc@
