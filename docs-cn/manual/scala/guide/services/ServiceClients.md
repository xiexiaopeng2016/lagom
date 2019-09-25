# 消费服务

我们已经了解了如何定义服务描述符以及如何实现它们，现在我们需要消费它们。服务描述符包含Lagom需要了解有关如何调用服务的所有信息，因此，Lagom能够为您实现服务描述符接口。

## 绑定服务客户端

消费服务的第一件事是创建服务的实现。Lagom提供了一个宏来执行此操作, 在[`ServiceClient`](api/com/lightbend/lagom/scaladsl/client/ServiceClient.html)类上, 名为`implement`。该`ServiceClient`由[`LagomServiceClientComponents`](api/com/lightbend/lagom/scaladsl/client/LagomServiceClientComponents.html)提供，其已经被`LagomApplication`实现，所以要创建一个从Lagom应用服务客户端，你只需要做以下事情:

@[implement-hello-client](code/ServiceClients.scala)

## 使用服务客户端

绑定客户端后，您现在可以在Lagom应用程序中的任何位置使用它。通常，通过将其传递到另一个组件(例如服务实现)的构造函数来完成，如果您使用Macwire，则会自动为您完成。这是从另一服务中消费一个服务的示例:

@[hello-consumer](code/ServiceClients.scala)

## 流服务客户端配置

使用流服务客户端时，Lagom将在内部使用一个WebSocket客户端，该客户端具有最大帧长(max frame length)参数。此参数限制通过WebSocket的消息的允许的最大大小。可以`application.conf`在客户端上进行配置，默认配置为:

@[web-socket-client-default](../../../../../service/core/client/src/main/resources/reference.conf)

此配置将影响服务客户端消费的所有流服务。当使用多个流服务时，不可能提供不同的配置。

请注意，相同的参数必须使用[Play服务器配置](https://www.playframework.com/documentation/2.6.x/ScalaWebSockets#Configuring-WebSocket-Frame-Length)在服务器端配置

## 断路器(Circuit Breakers)

[断路器](https://martinfowler.com/bliki/CircuitBreaker.html)是在分布式系统中用来提供稳定性并防止级联故障。这些应与服务之间的接口上的明智的(judicious)超时一起使用，以防止单个服务的故障导致其他服务中断。

例如，我们有一个与第三方Web服务交互的Web应用程序。
假设第三方容量已经超负荷，并且他们的数据库崩溃了。假定数据库以这种方式失败，其花费很长时间将错误交还给第三方Web服务。这样会使调用在很长一段时间后失败。回到我们的Web应用程序，用户已经注意到，他们提交的表单似乎花费了更长的时间。用户使用刷新按钮来完成他们所知道的操作，从而将更多请求添加到他们已经在运行的请求中。由于资源耗尽，这最终导致Web应用程序失败。

在Web服务调用中引入断路器会导致请求开始就快速失败，从而使用户知道出了问题并且不需要刷新请求。这也将故障行为限制为仅使用依赖于第三方的功能的那些用户，其他用户不再受到影响，因为没有资源耗尽。断路器还可以使精明的开发人员标记网站上使用该功能不可用的部分，或者在断路器打开时适当地显示一些缓存的内容。

断路器具有3种状态：

[[circuit-breaker-states.png]]

在正常操作期间，断路器处于**Closed**状态:

* 异常或调用超出配置的`call-timeout`增加故障计数器
* 成功将失败计数重置为零
* 当故障计数器达到`max-failures`数目时，断路器将跳转到打开状态

处于**Open**状态时:

* 所有调用快速失败，并带有`CircuitBreakerOpenException`
* 配置完成`reset-timeout`后，断路器进入半开状态

在**Half-Open**状态:

* 第一个调用尝试允许通过，不会很快失败
* 所有其他调用都快速失败, 有异常，就像在打开状态一样
* 如果第一个调用成功，则将断路器重置回闭合状态
* 如果第一个调用失败，则断路器再次跳至打开状态以进行另一个完全复位(full resetTimeout)

所有与Lagom服务客户端的服务调用都默认使用断路器。断路器是在客户端使用和配置的，但是粒度和配置标识符由服务提供商定义。默认情况下，一个断路器实例用于对另外的服务的所有调用(方法)。可以为每种方法设置唯一的断路器标识符，以便为每种方法使用单独的断路器实例。还可以通过在几个方法上使用相同的标识符对相关方法进行分组。

@[circuit-breaker](code/ServiceClients.scala)

在上面的示例中，默认标识符用于该sayHi方法，因为没有给出特定的标识符。默认标识符与服务名称相同，在此示例中即`"hello"`。该`hiAgain`方法将使用另一个断路器实例，因为`"hello2"`被指定为断路器标识符。

### 断路器配置

在客户端，您可以配置断路器。默认配置为:

@[circuit-breaker-default](../../../../../service/core/client/src/main/resources/reference.conf)

如果您自己未定义任何配置，则将使用该配置。用于配置断路器的设置包括您在断路器中期望的常规设置，如故障数量或应该打开电路的请求超时以及必须超时才能再次关闭电路的超时。在lagom中，有一个额外的设置来控制什么被认为是失败。

Lagom的客户端[[将所有4xx和5xx响应映射到异常|ServiceErrorHandling]]，而Lagom的断路器默认将所有异常视为故障。您可以通过将特定异常列入白名单来更改默认行为，以使它们不算作失败。有时您想要为给定的端点配置断路器，因此它会忽略某些异常。当连接到使用4xx HTTP状态代码为业务有效案例建模的服务时，这特别有用。例如，响应“ 404未找到”来响应特定请求可能是非失败案例。在这种情况下，您可以将其添加`"com.lightbend.lagom.scaladsl.api.transport.NotFound"`到断路器白名单中，这样就不会将其视为故障。即使NotFound异常不算作失败，客户端仍会抛出NotFound 调用服务导致的异常。

通过上面的"hello"示例，我们可以通过定义属性来调整配置，`application.conf`例如：

    lagom.circuit-breaker {

      # will be used by sayHi method
      hello.max-failures = 5

      # will be used by hiAgain method
      hello2 {
        max-failures = 7
        reset-timeout = 30s
      }

      # Change the default call-timeout
      # will be used for both sayHi and hiAgain methods
      default.call-timeout = 5s
    }

[Lightbend Monitoring](https://www.lightbend.com/products/monitoring)将提供Lagom断路器的指标，包括群集中所有节点的信息汇总视图。
