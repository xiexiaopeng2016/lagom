# 测试服务

## 运行测试

可以从sbt或从IDE运行测试。从您的IDE运行测试将特定于您的IDE，因此这里我们将重点介绍如何从sbt运行测试。

* 要运行所有测试，请运行`test`。
* 要仅运行一个测试类，请运行`testOnly`后跟类的名称，即`testOnly com.example.MyTest`。支持通配符，比如您可以说`testOnly *.MyTest`。
* 要仅运行覆盖已更改代码的测试，请运行`testQuick`。
* 要连续运行测试，请在命令前面加一个波浪号，即`~testQuick`。

## 测试库

您可以将任何测试框架与Lagom一起使用，流行的框架包括[ScalaTest](http://www.scalatest.org/) 和 [Specs2](https://etorreborre.github.io/specs2/)。如果您不确定要使用哪个或没有偏好，我们将使用ScalaTest来测试Lagom本身，我们将在此处进行记录。

除了测试框架外，Lagom还提供了一个用于测试常见Lagom组件的帮助程序库，称为Lagom testkit。

要使用首选的测试框架和Lagom testkit，您需要将它们添加到库依赖项中，如下所示:

@[test-dependencies](code/testing-services.sbt)

您可能想要在构建的多个位置使用ScalaTest，因此通常最好创建一个`val`保存值，这意味着您可以从需要的每个位置引用该val，而不必重新键入组id，工件ID和版本。可以这样完成：

@[scala-test-val](code/testing-services.sbt)

然后，您可以`libraryDependencies`通过简单地引用它来在您的计算机中使用它：

@[test-dependencies-val](code/testing-services.sbt)

使用Cassandra时，必须对测试进行分叉，可以通过在项目的构建中添加以下内容来启用测试:

@[fork](code/testing-services.sbt)

## 如何测试一项服务

Lagom支持独立地为一个服务编写功能测试。服务在服务器中运行，在测试中您可以使用它的服务客户端，即调用服务API, 与它交互。这些实用程序在ServiceTest中定义。

这是一个简单的测试结果:

@[hello-service-spec](code/TestingServices.scala)

关于此代码，有几点要注意:

* 该测试使用ScalaTest的[异步测试支持](http://www.scalatest.org/user_guide/async_testing)。实际的测试本身会返回一个future，ScalaTest确保可以正确处理该future。
* `withServer`接受三个参数。第一个是设置参数，可用于配置环境的设置方式，例如，可用于启动Cassandra。第二个是[LagomApplication](api/com/lightbend/lagom/scaladsl/server/LagomApplication.html)的构造函数，这是我们在其中构造应用程序的地方，第三个是要运行的块，它使用启动的服务器并运行实际测试。
* 当构造`LagomApplication`时，我们将混入`LocalServiceLocator`。这提供了一个本地服务定位器，该定位器将仅解析我们的应用程序自身正在运行的服务，并且这是我们构造的服务客户端如何知道在哪里可以找到正在运行的服务的地方。
* 在测试回调中，我们实现了一个服务客户端，然后可以使用它与我们的服务进行对话。

上面的规范将为每个测试启动服务器，这通常很方便，因为它可以保证每个测试之间的干净状态。但是，有时为每个测试启动服务器的代价过高，尤其是在涉及数据库时。在这些情况下，最好在套件的所有测试之间共享服务器。为此，`startServer`可以代替使用，`stop`在after suite回调中调用:

@[hello-service-spec-shared](code/TestingServices.scala)

必须通过在`LagomApplication`构造函数回调中覆盖存根或模拟实现来替换对其他服务的依赖。如果我们正在针对`HelloService`编写测试，并且该测试依赖于`GreetingService`则必须创建`GreetingService`的存根实现，它可用于测试，而无需运行真正的greeting服务。它可能看起来像这样:

@[stub-services](code/TestingServices.scala)

服务器默认运行[[pubsub|PubSub]]，禁用[[集群|Cluster]] 和 [[持久性|PersistentEntity]]功能。您可能要在`Setup`中启用集群:

@[enable-cluster](code/TestingServices.scala)

如果您的服务需要[[持久性|PersistentEntity]]，则需要显式启用它。这可以通过启用Cassandra或JDBC来完成，具体取决于您的服务使用哪种持久性。无论如何，Lagom持久性都需要集群，因此启用一个或另一个集群时，集群也会自动启用。

您不能同时启用两者(Cassandra和JDBC)进行测试，如果您在写和读方面混入了持久性，则可能会出现问题。如果将Cassandra用于写端，将JDBC用于读端，则只需启用Cassandra。

要启用Cassandra持久性:

@[enable-cassandra](code/TestingServices.scala)

要启用JDBC持久性:

@[enable-jdbc](code/TestingServices.scala)

无法显式启用或禁用[[pubsub|PubSub]]。启用集群后(通过启用Cassandra或JDBC显式或可传递)，pubsub将可用。

## 如何在测试中使用TLS

要打开测试中使用的`TestServer`上的SSL端口，可以使用`withSsl`命令启用S​​SL支持:

```java
Setup.defaultSetup.withSsl()
```

启用S​​SL将自动打开一个新的随机端口，并在TestServer上提供的`javax.net.ssl.SSLContext`。Lagom目前不提供任何允许将请求发送到HTTPS端口的客户端工厂。您应该使用Play-WS，Akka-HTTP或Akka-gRPC创建HTTP客户端。然后，使用`testServer`实例提供的`httpsPort`和`sslContext`来发送请求。请注意，提供的`SSLContext`是由Lagom的testkit构建的，以信任`testServer`证书。最后，由于服务器证书是为您颁发的，因此必须确保生成的请求中包含该证书，否则服务器可能会拒绝并导致请求失败。目前，无法为测试服务器设置不同的SSL证书。

@[tls-test-service](../../../../../testkit/scaladsl/src/test/scala/com/lightbend/lagom/scaladsl/testkit/TestOverTlsSpec.scala)




## 如何测试几种服务

Lagom将为编写涉及多个交互服务的集成测试提供支持。此功能[尚未实现](https://github.com/lagom/lagom/issues/38)。

## 如何测试流式请求/响应

假设我们有一个具有流请求和/或响应参数的服务。例如`EchoService`这样的:

@[echo-service](code/TestingServices.scala)

在为此编写测试时，[Akka Streams TestKit](https://doc.akka.io/docs/akka/2.6/stream/stream-testkit.html?language=scala#streams-testkit)非常有用。我们将Streams TestKit与Lagom `ServiceTest`实用程序一起使用：

@[echo-service-spec](code/TestingServices.scala)

在[Akka Streams TestKit](https://doc.akka.io/docs/akka/2.6/stream/stream-testkit.html?language=scala#streams-testkit)的文档中阅读有关它的更多信息。

## 如何测试持久性实体



[[持久性实体|PersistentEntity]]可以在上述服务测试中使用。除此之外，您还应该使用[PersistentEntityTestDriver](api/com/lightbend/lagom/scaladsl/testkit/PersistentEntityTestDriver.html)编写单元测试，这将运行在`PersistentEntity`不使用数据库的情况下。

[[持久实体|PersistentEntity#Unit-Testing]]文档中对此进行了描述。
