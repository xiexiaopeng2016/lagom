# 实现服务

服务是通过提供实现服务描述符特质来实现的，实现该描述符指定的每个调用。

例如，这是`HelloService`描述符的实现:

@[hello-service-impl](code/ServiceImplementation.scala)

如您所见，该`sayHello`方法是使用[`ServiceCall`](api/com/lightbend/lagom/scaladsl/api/ServiceCall$.html)的`apply`工厂方法实现的。这需要一个函数，`Request => Future[Response]`并返回一个[`ServiceCall`](api/com/lightbend/lagom/scaladsl/api/ServiceCall.html), 其`invoke`方法仅简单地委托给该函数。这里要意识到的重要一点是，`sayHello`自身不会执行调用，它只会返回要执行的服务调用。这样做的好处是，在将调用与其他横切关注点, 例如身份验证, 进行组合时，可以使用基于普通功能的组合轻松完成此操作。

让我们再看一下[`ServiceCall`](api/com/lightbend/lagom/scaladsl/api/ServiceCall.html)接口：

@[service-call](code/ServiceDescriptors.scala)

它将接受请求，并以[`Future`](https://www.scala-lang.org/api/2.12.x/scala/concurrent/Future.html)形式返回响应。如果您从未见过`Future`，这个值可能要等到以后才能使用。当API返回future时，这个值可能还没有计算出来，但API承诺会在将来的某个时候计算该值。由于这个值还没有计算出来，所以您不能立即与它进行交互。不过，您可以使用`map`和`flatMap`方法附加回调，将promise转换为一个新值的promise。`Future` 它的map和flatMap方法是在Scala中进行反应式编程的基本构建块，它们允许您的代码异步执行，不等待事情发生，而是附加回调，以响应正在完成的计算。

当然，简单的hello world计算不是异步的，它所需要做的就是构建一个String，然后立即返回。在这种情况下，我们需要将结果包装在中`Future`。这可以通过调用`Future.successful()`来完成，该方法返回具有立即可用值的Future。

## 使用流

当请求和响应主体是严格时，使用它们就很简单。但是，如果它们是流式的，则需要使用Akka流来使用它们。让我们看一下如何实现[[服务描述符|ServiceDescriptors#Streamed-messages]]示例中的某些流服务调用。

该`tick`服务调用是否会返回一个`Source`，其在指定的时间间隔发送消息。Akka streams为此类流提供了一个有用的构造函数:

@[tick-service-call](code/ServiceImplementation.scala)

前两个参数是发送消息之前的延迟以及发送消息的时间间隔。第三个参数是应该在每个tick上发送的消息。使用间隔为`1000`和`tick`请求消息调用此服务将导致返回一个流，其每秒发送一条`tick`消息。

sayHello可以通过将Source名称的传入映射为打招呼来实现流服务调用：
一个流式的`sayHello`服务调用可以被实现, 通过映射传入`Source`类型的names来说hello:

@[hello-service-call](code/ServiceImplementation.scala)

当您`map` 一个 `Source`时，您会得到一个新的`Source`，它将映射转换应用于传入的`Source`产生的每个消息。

这些使用流的示例显然是微不足道的。有关[[发布-订阅|PubSub]]和[[持久化读取侧|ReadSide]]的部分，显示了在Lagom中使用流的真实示例。

## 处理消息头

有时您可能需要处理请求头，或向响应头添加信息。`ServiceCall`提供了`handleRequestHeader`和`handleResponseHeader`方法允许您执行此操作，但是不建议您直接实现这些，而应该使用[`ServerServiceCall`](api/com/lightbend/lagom/scaladsl/server/ServerServiceCall.html)。

`ServerServiceCall`是一个扩展`ServiceCall`的接口，并提供一个附加的方法`invokeWithHeaders`。这与常规`invoke`方法不同，因为除了`Request`参数之外，它还接受[`RequestHeader`](api/com/lightbend/lagom/scaladsl/api/transport/RequestHeader.html)参数。而且不是返回一个`Future[Response]`，而是返回一个`Future[(ResponseHeader, Response)]`。因此，它允许您处理请求头，并发送自定义响应头。`ServerServiceCall`实现 `handleRequestHeader` 和 `handleResponseHeader` 方法，以便当Lagom调用该`invoke`方法时，它将委托给 `invokeWithHeaders`方法。

[`ServerServiceCall`](api/com/lightbend/lagom/scaladsl/server/ServerServiceCall$.html)的同伴对象提供一个工厂方法用于创建`ServerServiceCall`，可以同时处理请求头和响应头。能够创建一个`ServerServiceCall`不能使用消息头的方法似乎有违直觉，但是这样做的原因是为了协助服务调用的组合，在这种情况下，组合服务调用可能希望组合这两种类型的服务调用。

这是使用消息头的示例：

@[server-service-call](code/ServiceImplementation.scala)

## 服务调用组合

在某些情况下，您可能希望编写涉及安全性或日志记录等跨领域问题的服务调用。在Lagom中，这是通过显式地组合服务调用来实现的。下面是一个简单的日志服务调用:

@[logging-service-call](code/ServiceImplementation.scala)

这使用来自`ServerServiceCall`的`compose`方法，该方法采用一个带有请求头的回调，并返回一个服务调用。

如果我们要实现`HelloService`用于日志记录，则可以这样使用它：

@[logged-hello-service](code/ServiceImplementation.scala)

另一个常见的横切关注点(cross cutting concern)是身份验证。假设您有一个用户存储接口:

@[user-storage](code/ServiceImplementation.scala)

您可以像这样使用它来实现经过身份验证的服务调用：

@[auth-service-call](code/ServiceImplementation.scala)

这次，由于用户的查询是异步的，因此我们使用`composeAsync`，它允许我们异步返回服务调用以处理服务。另外，我们不只是接受服务调用，还接受用户对服务调用的函数。这意味着服务调用可以访问用户：

@[auth-hello-service](code/ServiceImplementation.scala)

注意，与其他框架不同的是，在其他框架中，user对象可以使用线程局部变量或过滤器在非类型化映射中传递，user对象是显式传递的。
如果您的代码需要访问user对象，则不可能出现配置错误，而您忘记将过滤器放在适当的位置，则代码将无法编译。

通常，您会希望将多个服务调用组合在一起。与注释相比，这正是基于函数的组合的强大之处。由于服务调用只是常规方法，因此您可以简单地定义一个的新方法将它们组合在一起，如下所示：

@[compose-service-call](code/ServiceImplementation.scala)

在hello服务中使用它:

@[filter-hello-service](code/ServiceImplementation.scala)
