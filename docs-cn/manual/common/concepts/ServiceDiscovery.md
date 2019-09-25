# 注册和发现服务

为了具有弹性和可伸缩性，系统必须支持位置透明性。这使您可以在多个主机上运行同一微服务的实例，在发生故障时将实例从一台主机移到另一台主机，并随着负载的变化上下扩展主机的数量。在这样的反应式系统中，微服务实例不断移动，产生和死亡，客户端和其他微服务都需要一种方法来定位可用的服务实例。

Lagom在开发过程中为您的微服务系统提供以下功能：

* [服务注册](#Service-registration)
* [客户端服务发现](#Client-side-service-discovery)
* [服务器端服务发现](#Server-side-service-discovery)

*Note*: 此页面使用HTTP示例，但是这些概念适用于任何类型的通信，例如基于TCP的二进制。

## 服务注册

服务注册中心与微服务实例协作以维护最新的查找表。该表包括每个可用微服务实例的主机和端口。随着负载的变化，系统可以在任何位置生成或销毁实例，同时继续满足请求。您可以设计一个系统来启用微服务自注册(self-register)，也可以使用[第3方注册服务](https://microservices.io/patterns/3rd-party-registration.html)。

在启动Lagom微服务实例时，注册者将在服务注册表上注册微服务的名称、URL和可定位服务描述符的名称，以便可以定位它们。关闭服务实例的电源时，注册者也必须更新服务注册表。Lagom的[[开发人员环境|DevEnvironment]]提供了服务注册表和注册器的实现，因此您可以在本地运行微服务。

<!---The following illustrates service registration. (TBA) --->

许多可用的技术都提供服务注册功能。您将需要选择和/或开发服务定位器，以便在部署环境中运行服务(例如，参见[Lagom ZooKeeper服务定位器](https://github.com/jboner/lagom-service-locator-zookeeper))。您可能需要找出一种方法来将Lagom服务与注册者联系。

## 客户端服务发现

来自Bonér的[反应式微服务体系结构：分布式系统的设计原理](https://info.lightbend.com/COLL-20XX-Reactive-Microservices-Architecture-RES-LP.html)

> 一旦存储了有关每个服务的信息，便可以通过服务注册表使服务可用，该服务可以使用服务注册表来查找信息 -使用称为客户端服务发现的模式。

Lagom为每个服务描述符创建服务客户端，以便应用程序可以与Lagom服务进行交互。假设非Lagom应用程序想要使用hello服务。它可以使用欢迎服务客户端并仅调用`hello`方法。欢迎服务客户端将使用服务注册表在`welcome`可用的地方找到有效的URL并履行请求。这种方法需要使用Lagom提供的代码。在生产中，插入到您的服务中的服务定位器将成为参与此客户端发现的元素。有关更多信息，请参见[[与非Lagom服务集成|IntegratingNonLagom]]。

<!--- The following diagram illustrates client-side service discovery. (TBA) --->

## 服务器端服务发现

来自Bonér的[反应式微服务体系结构：分布式系统的设计原理](https://info.lightbend.com/COLL-20XX-Reactive-Microservices-Architecture-RES-LP.html)

> 另一种策略是使用称为服务器端服务发现的模式将信息存储并维护在负载均衡器中。

如果不能在每个客户端上都嵌入服务定位器，则可以使用服务器端服务发现。此模式使用服务网关来允许客户端使用服务描述符注册提供的端点。在此模型中，浏览器仅需要知道服务网关在哪里。当使用服务器端发现时，只有当服务描述符的调用被添加到ACL中时，才能访问该服务描述符上的调用。

例如，浏览器可以通过`/hello/steve`从服务网关请求路径来向用户显示问候消息。服务网关将知道哪个服务提供了该终结点，并会向服务注册表询问该服务的实例。服务注册表将以可以满足请求的实例的主机和端口作为响应。最后，服务网关将执行请求，并将结果返回到浏览器。

<!--- The following diagram illustrates server-side service discovery. (TBA) -->

为了简化服务器端服务发现的测试，Lagom开发环境将启动所有服务以及服务注册表和服务网关。
