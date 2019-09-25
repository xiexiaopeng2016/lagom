# Scala 组件

这是可用于构建应用程序蛋糕的可用组件的列表。该列表包含Lagom，Play和其他库提供的组件。您可以开发自己的组件，并使用这些组件代替此列表中提供的组件，只要合同已履行即可。

#####  服务组件

| -------------------- | ----------- |
| [LagomServerComponents](api/com/lightbend/lagom/scaladsl/server/LagomServerComponents.html) | Lagom服务的主要组件。请参阅[[定义自己的组件|DependencyInjection#Defining-your-own-components]]。|
| [LagomServiceClientComponents](api/com/lightbend/lagom/scaladsl/client/LagomServiceClientComponents.html) | 任何Lagom服务或使用Lagom服务的应用程序的主要组件。请参阅[[Lagom中的依赖注入|DependencyInjection]]和[[绑定服务客户端|ServiceClients#Binding-a-service-client]]。|

##### 持久性和群集组件
| -------------------- | ----------- |
| [ClusterComponents](api/com/lightbend/lagom/scaladsl/cluster/ClusterComponents.html) |  registers the node to the Akka Cluster. The Akka Cluster is required by Pub-Sub and Persistent Entity support and you will rarely need to use it explicitly. See [[Cluster]]|
| [PubSubComponents](api/com/lightbend/lagom/scaladsl/pubsub/PubSubComponents.html) | provides [[Publish-Subscribe|PubSub#Publish-Subscribe]]. This requires `ClusterComponents`. See [[Publish-Subscribe|PubSub#Publish-Subscribe]]|
| [CassandraPersistenceComponents](api/com/lightbend/lagom/scaladsl/persistence/cassandra/CassandraPersistenceComponents.html) |  provides both Read-Side and Write-Side components for Cassandra-backed CQRS. It provides `ReadSideCassandraPersistenceComponents` and `WriteSideCassandraPersistenceComponents` which you might want to use in isolation. See [[PersistentEntityCassandra]] for more info. |
| [JdbcPersistenceComponents](api/com/lightbend/lagom/scaladsl/persistence/jdbc/JdbcPersistenceComponents.html) | provides both Read-Side and Write-Side components for Cassandra-backed CQRS. It provides `ReadSideJdbcPersistenceComponents`and `WriteSideJdbcPersistenceComponents` which you might want to use in isolation. See [[PersistentEntityRDBMS]]. |
| [ProjectionComponents](api/com/lightbend/lagom/scaladsl/projection/ProjectionComponents.html) | provides `projections` to query the status and stop and start your projection workers. See [[Projections]]. |

##### Broker API Components
| -------------------- | ----------- |
| [LagomKafkaClientComponents](api/com/lightbend/lagom/scaladsl/broker/kafka/LagomKafkaClientComponents.html) | provide a Kafka implementation for the Broker API so that you Service can subscribe to a Kafka topic. See[[KafkaClient|KafkaClient#Subscriber-only-Services]]. |
| [LagomKafkaComponents](api/com/lightbend/lagom/scaladsl/broker/kafka/LagomKafkaComponents.html) | provides a Kafka implementation for the Broker API so that you Service can `publish` to a Kafka topic. This component includes `LagomKafkaClientComponents` so if you mix this one in, you will be able to `publish` and `subscribe`. See [[Kafka Client|KafkaClient#Dependency]]. |
| [TestTopicComponents](api/com/lightbend/lagom/scaladsl/testkit/TestTopicComponents.html) | provides stubbing tools to test services that use the Broker API. This is meant to be used only on Applications you build on your tests. See [[Testing publish|MessageBrokerTesting#Testing-publish]] |

##### Service Locator Components
| -------------------- | ----------- |
| [LagomDevModeServiceLocatorComponents](api/com/lightbend/lagom/scaladsl/devmode/LagomDevModeServiceLocatorComponents.html) | provides the dev mode service locator. This is meant to be used by Lagom Services and other applications such as Play Apps that want to interact with Lagom Services in Dev Mode. See the [scaladocs](api/com/lightbend/lagom/scaladsl/devmode/LagomDevModeServiceLocatorComponents.html) for more details. |
| [LagomDevModeComponents](api/com/lightbend/lagom/scaladsl/devmode/LagomDevModeComponents.html) | provides the dev mode service locator and registers the services with it in dev mode. See [[Wiring together a Lagom application|DependencyInjection#Wiring-together-a-Lagom-application]].|
| [StaticServiceLocatorComponents](api/com/lightbend/lagom/scaladsl/client/StaticServiceLocatorComponents.html)| provides a Service Locator that always resolves the same URI for a given Service name. |
| [ConfigurationServiceLocatorComponents](api/com/lightbend/lagom/scaladsl/client/ConfigurationServiceLocatorComponents.html)| provides a Service Locator based on `application.conf` files. See [[Using static values for services and Cassandra to simulate a managed runtime|ProductionOverview#Using-static-values-for-services-and-Cassandra]].|
| [RoundRobinServiceLocatorComponents](api/com/lightbend/lagom/scaladsl/client/RoundRobinServiceLocatorComponents.html)| provides a Service Locator that applies a Round Robin over the passed in sequence of URI. |
| [CircuitBreakerComponents](api/com/lightbend/lagom/scaladsl/client/CircuitBreakerComponents.html)| implementors of a Service Locator will need to extend this to reuse the Circuit Breaker config provided by Lagom.|
| [AkkaDiscoveryComponents](api/com/lightbend/lagom/scaladsl/akka/discovery/AkkaDiscoveryComponents.html)| implementors of a Service Locator based on Akka Discovery. Available as [[opt-in dependency|AkkaDiscoveryIntegration]]. It is the recommended implementation for production specially for users targeting Kubernetes and DC/OS (Marathon).|

##### Third party Components

You can mix in `Components` from other frameworks or libraries, for example:

 * [AhcWSComponents](https://www.playframework.com/documentation/2.6.x/api/scala/index.html#play.api.libs.ws.ahc.AhcWSComponents): provides a `WSClient` based on an Async HTTP Client.
 * [DBComponents](https://www.playframework.com/documentation/2.6.x/api/scala/play/api/db/DBComponents.html)
 * [HikariCPComponents](https://www.playframework.com/documentation/2.6.x/api/scala/play/api/db/HikariCPComponents.html)

Lagom inherits all Components provided by Play. You can find the complete list of inherited `Components` by searching for _components_ in [Play docs](https://www.playframework.com/documentation/2.6.x/api/scala/index.html).
