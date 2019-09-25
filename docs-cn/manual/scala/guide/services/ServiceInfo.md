# 服务元数据

服务元数据, 也称为`ServiceInfo`, 包括名称和服务ACL的集合。在大多数情况下，元数据都是自动计算的，您无需查看甚至提供它。

Lagom支持以下几种方案:

1. 当您创建Lagom服务, 并[[将应用程序连接在一起|DependencyInjection#Wiring-together-a-Lagom-application]]时， Lagom会将`name`和ACL捆绑为一个`ServiceInfo`。
2. 当您消费Lagom服务并通过混入`LagomServiceClientComponents`方式[[绑定客户端|ServiceClients#Binding-a-service-client]]时， Lagom不会在背后连接ServiceInfo，您必须以编程方式提供一个。
3. 最后的方案是客户端应用程序不使用Guice, 并通过[[Lagom Client Factory|IntegratingNonLagom]]连接到Lagom。在这个方案中，Lagom还将代表您创建元数据。


## 服务名称和服务ACL

服务彼此交互。这种交互需要每个服务在扮演另一个服务的客户端时标识自己。需要此标识时，默认使用`ServiceInfo`的名称。举个例子`HelloService`:

@[service-name](code/ServiceInfo.scala)

如果Greetings服务打包了`HelloService`，并且Greetings服务正在调用I18n服务(不在代码段中)，则这些调用将包括标识`hello`, 因为它是`HelloService`的名称(请参阅`named("hello")`)。

服务可以在服务网关中发布acl，以列出服务提供的端点。这些acl将允许您通过服务网关开发[[服务器端服务发现|ServiceDiscovery#Server-side-service-discovery]]。

@[service-acls](code/ServiceInfo.scala)

在此示例中，`UsersService`开发人员设置`withAutoAcl`为`true`。这指示Lagom从每个调用的`pathPattern`生成服务ACL。在此示例中，将为`/api/users/login`创建一个ACL 。部署时，您的工具应遵守这些规范，并确保API网关设置正确。
