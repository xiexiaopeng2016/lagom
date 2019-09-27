# 服务错误处理

Lagom提供了许多不同的机制来控制和定制服务之间处理和报告错误的方式。

Lagom内置的错误处理的设计背后有许多原则：

* 在生产中，除非Lagom服务知道这样做是安全的，否则它永远不应该向另一个服务提供它遇到的错误的详细信息。出于安全原因，攻击者可以使用未经审查的错误消息来获取有关服务如何实现的详细信息。在实践中，这意味着有许多内置的异常，Lagom认为是安全的，可以返回其详细信息，其余的则不返回任何内容。
* 在开发中，通过传输发送完整的错误消息很有用。当服务在开发中运行时，Lagom将尝试发送有关异常的有用信息。
* 如果可能，Lagom将在服务端抛出错误时尝试在客户端重建错误。因此，如果服务器端抛出异常，表明它无法序列化某些内容，则客户端代码应接收相同的异常。
* 如果可能，应将异常映射到惯用的协议响应代码，例如HTTP 4xx和5xx状态代码以及WebSocket错误关闭代码。

如果您使用Lagom消费服务(在Lagom或第三方堆栈中实现)，则客户端Lagom提供的服务会将响应中的值在4xx和5xx范围内的状态码映射到异常。这会影响客户端用来连接到该端点的[[断路器|ServiceClients#Circuit-Breakers]]。默认情况下，Lagom断路器会将任何异常都视为失败，但该行为是[[可配置的|ServiceClients#Circuit-Breaker-Configuration]]。因此，4xx和5xx将映射到异常，但是您可以将那些不应该跳闸断路器得异常列入白名单。

## 异常序列化器

Lagom提供了一个[`ExceptionSerializer`](api/com/lightbend/lagom/scaladsl/api/deser/ExceptionSerializer.html)特质，该特质允许将异常序列化为某种形式，例如JSON，并选择错误代码。它还允许从错误代码及其序列化形式重新创建异常。

异常序列化器将异常转换为[`RawExceptionMessage`](api/com/lightbend/lagom/scaladsl/api/deser/RawExceptionMessage.html)。原始异常消息包含状态代码，该状态代码将对应于HTTP状态代码或WebSocket关闭代码，消息主体和协议描述符，以说明消息的内容类型是什么 - 在HTTP中，它将转换为响应的`Content-Type`报头。

Lagom提供的默认异常序列化器使用Play JSON将异常序列化为JSON。这个异常序列化器实现了上述准则 - 如果它是的[`TransportException`](api/com/lightbend/lagom/scaladsl/api/transport/TransportException.html)子类, 它将仅返回该异常的详细信息，除非在开发中。

在`TransportException`子类中有几个有用的构建，您可以使用它们，其中包括[`NotFound`](api/com/lightbend/lagom/scaladsl/api/transport/NotFound.html) 和 [`PolicyViolation`](api/com/lightbend/lagom/scaladsl/api/transport/PolicyViolation.html)。

Lagom通常能够将这些异常传递给客户端。您也可以直接实例化`TransportException`并使用它，或者可以定义`TransportException`的子类，但是请注意，Lagom不会将该子类扔给客户端，因为它不会知道，除非您提供自定义的异常序列化器来处理。
