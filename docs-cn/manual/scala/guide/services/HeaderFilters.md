# 头部过滤器

在Lagom中，您可以将`HeaderFilter`添加到服务描述符中。在`HeaderFilter`中，您通常会处理协议协商或身份验证。

单个`HeaderFilter`实现可以转换一个请求离开客户端或进入服务器和一个响应离开服务器和进入客户端。
可以转换离开客户端或进入服务器的请求和离开服务器并进入客户端的响应。这是一个使用`User-Agent`头部读取服务名称的示例：

@[user-agent-header-filter](../../../../../service/scaladsl/api/src/main/scala/com/lightbend/lagom/scaladsl/api/transport/HeaderFilter.scala)

如果未指定，这个[`UserAgentHeaderFilter`](api/com/lightbend/lagom/scaladsl/api/transport/UserAgentHeaderFilter$.html)是任何Lagom服务将默认使用的`HeaderFilter`。它使用一个`ServicePrincipal`， 其用服务名称标识客户端。

在`UserAgentHeaderFilter`中，位于`transformClientRequest`的代码将在准备一个客户端调用去添加一个`User-Agent`头部时被调用，假如请求中指定了一个[`ServicePrincipal`](api/com/lightbend/lagom/scaladsl/api/security/ServicePrincipal.html)。请注意，默认情况下，Lagom在它发出一个请求时会将当前服务名称作为`ServicePrincipal`自动传递。在服务器端`transformServerRequest`将用于读取`User-Agent`头部并将该值设置为请求的[Principal](https://docs.oracle.com/javase/8/docs/api/java/security/Principal.html)。

请记住，头部过滤器仅应用于处理涉及cross cutting的协议，仅此而已。例如，您可能有一个报头过滤器，它描述当前经过身份验证的用户如何通过HTTP协议进行通信(例如，通过添加一个用户报头)。涉及Cross cutting的领域，如认证和验证，不应该在头过滤器处理，而应该使用[[服务调用组合| ServiceImplementation#Service-call-composition]]来处理。

## 头部过滤器组成

每个服务`Descriptor`只能有一个`HeaderFilter`。为了一次使用多个过滤器，您可以使用`HeaderFilter.composite`组合它们，这将返回一个`HeaderFilter`，它链接所有您组合在一起的`HeaderFilter`。组合时，顺序很重要，因此在发送消息头时，组合的过滤器会使用你提供的顺序，在接收消息头时，将以相反的顺序使用过滤器。因此，如果我们具有以下过滤器：

@[verbose-filter](code/HeaderFilters.scala)

我们用名称`Foo` 和`Bar`注册了其中两个：

@[header-filter-composition](code/HeaderFilters.scala)

然后调用该服务，那么我们将在服务器输出日志中获得以下信息：

```
[debug] Bar - transforming Server Request
[debug] Foo - transforming Server Request
[debug] Foo - transforming Server Response
[debug] Bar - transforming Server Response
```
