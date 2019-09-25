#内部和外部通信

正如Lagom[[Lagom设计哲学|LagomDesignPhilosophy]]中所讨论的，服务应该是隔离的和自治的。这样的服务通过在网络上发送消息来相互通信(服务间)。为了实现性能和弹性，您经常要运行相同服务的多个实例，通常是在不同的节点上，而且这种服务内部通信也要通过网络进行。此外，第三方和/或旧式系统也可能会消费或为您的微服务系统提供信息。

以下主题更详细地讨论了这些通信路径:

* [微服务系统内的通信](#Communication-within-a-microservices-system)
* [与微服务系统外部的各方进行通信](#Communication-with-parties-outside-of-a-microservices-system)

## 微服务系统内的通信

尽管原则上相似，但是服务间和服务内通信具有非常不同的需求，并且Lagom提供了多种实现选项。服务间通信必须使用松耦合的协议和消息格式来保持隔离和自治。协调不同服务之间的更改可能是困难和昂贵的。您可以利用以下优势在系统中实现此目的：

服务调用（同步或异步（流））允许服务使用已发布的API和标准协议（HTTP和WebSockets）相互通信。

将消息发布到代理（例如Apache Kafka）会进一步使通信脱钩。Lagom的Message Broker API提供了至少一次的语义。如果新实例开始发布信息，则其消息将添加到先前发出的事件中。如果新实例订阅了主题，则它们将接收过去，现在和将来的所有事件（只要已订阅）。


* [[服务调用|ServiceDescriptors]], 同步或异步(流)都允许服务使用已发布的api和标准协议(HTTP和WebSockets)彼此通信。

* 将消息发布到[[代理|MessageBroker]], 例如Apache Kafka, 会进一步使通信脱钩. Lagom的[[Message Broker API|MessageBrokerApi]]提供了至少一次的语义。如果新实例开始发布信息，则其消息将添加到先前发出的事件中。如果新实例订阅了主题，则它们将接收过去，现在和将来的所有事件(只要已订阅)。


单个服务(统称集群)的节点需要较少的解耦。它们共享相同的代码, 作为一个集合, 并由一个团队或个人管理。因此，服务内通信可以利用开销较小，性能更好的机制。例如：

* 许多Lagom组件在内部使用[Akka remoting](https://doc.akka.io/docs/akka/2.6/general/remoting.html)处理，您可以在服务中直接使用它。

* [[分布式发布-订阅|PubSub]]可用于节点之间的低延迟，最多一次消息传递。限制包括:
<ul><ul>
<li>网络中断可能导致消息丢失。</li>
<li>当新实例启动，加入集群并进行订阅时，它将不会接收在订阅之前发送的消息。</li>
</ul></ul>

* 数据库和其他持久性存储可以看作是服务节点通信的另一种方式。对于使用持久实体的微服务，Lagom鼓励用[[事件流|ES_CQRS]]，它还利用异步通信的优势并通过事件日志提供保证。

该图说明了分布在三个服务器上的Lagom系统中服务间和服务内通信中的每种类型。在该示例中，订购服务发布一个或多个Kafka主题，而用户服务订阅以消费信息。用户服务使用Akka remoting处理与其他用户服务实例(集群成员)进行通信。运送服务和用户服务通过在服务呼叫中流式传输信息来交换信息。
[[ServiceCommunication.png]]

## 与微服务系统外部的各方进行通信

Lagom提倡使用异步通信，也不阻止在必要时使用同步通信。第三方可以异步地从发布到代理API的Lagom服务获取数据，并享受至少一次的保证。Lagom服务还公开了一个API，供第三方同步交换数据。通常将其映射到HTTP。Lagom服务API还通过websocket支持将数据流传输到外部客户端。查阅[[ServiceDescriptors]]了解更多信息。

与外部世界的交互可能意味着客户端, 比如浏览器、移动应用程序或物联网设备通过互联网使用这些服务。在使用标准HTTP或WebSocket时，通常此类客户端不会直接与单个服务进行通信。通常，网络边界充当外围，而控制良好的通信点充当外部世界与内部世界之间的中介。在Lagom，此通信点是服务网关。将您的微服务系统想象成中世纪的城镇，四周环绕一堵墙，一扇门是进出的唯一途径。围墙内部的通信是免费和直接的，但与外部世界的通信必须通过服务网关进行，如下图所示。
[[ExtraSystemCommunication.png]]


<!---For example, in the following diagram (see slide 5), notice the microservices running in a cluster on separate nodes (JVMs). The microservices in the cluster communicate with each other. Outside the cluster, a Service Gateway, a message broker, and other services also exchange messages. You can choose the type of communication appropriate for each service, whether that is: WebSockets, Akka pub-sub, or the Kafka message broker, and for services that need persistence, event streams. In the example, where all communication is asynchronous, failures or latency will not prevent any individual service from doing its job. -->
