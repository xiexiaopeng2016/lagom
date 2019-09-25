# Lagom的依赖注入

在用Lagom构建服务时，您的代码将依赖于Lagom API和其他需要在运行时通过具体实现来满足的服务。
通常，具体的实现在开发、测试和生产环境之间会有所不同，因此不要将您的代码紧密耦合到具体的实现类很重要。
一个类可以使用其依赖的抽象类型声明构造函数参数 --- 通常由Scala中的traits表示 --- 从而允许在构造类时提供具体的实现。这种模式称为"依赖注入"，是Lagom应用程序组装方式的基础。阅读"使用MacWire在Scala中进行依赖注入"，以获得有关此模式及其好处的更多背景资料。

反过来，您的服务的依赖在其他API中有其自己的依赖。综上所述，它们形成了一个依赖图，必须在应用程序启动时对其进行构造。此过程称为"wiring"您的应用程序，通过创建包含初始化代码的LagomApplication的子类来执行。

常见的情况是，相互依赖的类聚在一起形成一个更大的逻辑组件。以反映这些分组的方式对连接代码进行模块化可能会很有用。在Scala中，您可以通过定义一个"组件"特质来实现这一点，该特质包含一系列延迟val声明，这些声明实例化组件提供的接口的具体实现。它们的某些构造函数参数可以声明为抽象方法，指示由另一个组件或您的应用程序本身必须提供的一个依赖。然后，您的应用程序可以扩展多个组件，以将它们混合在一起成为一个完全组装的依赖关系图。如果您没有满足服务中包含的组件的所有已声明需求，那么它将无法编译。

在"[Scala中的依赖注入:指南](https://di-in-scala.github.io/#modules)"中，这种将组件混合在一起以形成应用程序的方法称为“薄蛋糕模式(thin cake pattern)”。该指南还介绍了[Macwire](https://di-in-scala.github.io/#macwire)，我们将在后续步骤中使用。

## 将Lagom应用程序连接在一起

在[[服务描述符|ServiceDescriptors]]和[[实现服务|ServiceImplementation]]部分中，我们创建了服务及其实现，现在，我们要绑定它们并使用它们创建一个Lagom服务器。Lagom的Scala API建立在Play框架之上，并使用Play的[编译时依赖注入支持](https://www.playframework.com/documentation/2.6.x/ScalaCompileTimeDependencyInjection)将Lagom应用程序连接在一起。Play的编译时依赖注入支持基于我们刚刚描述的薄蛋糕模式。

创建应用程序之后，您将使用Lagom提供的组件并从中获取依赖项(例如Akka Actor System或`CassandraSession`用于读取侧处理器访问数据库）。尽管这不是绝对必要的，但是我们建议您使用[Macwire](https://github.com/adamw/macwire)来辅助将依赖项连接到代码中。Macwire提供了一些非常轻量级的宏，这些宏可以找到要创建的组件的依赖，因此您不必自己手动将它们连接在一起。通过将以下内容添加到您的服务实现依赖中，可以将Macwire添加到您的服务中：

@[macwire](code/macwire.sbt)

构建应用程序蛋糕，然后将您的代码连接到其内的最简单方法是创建一个扩展了[`LagomApplication`](api/com/lightbend/lagom/scaladsl/server/LagomApplication.html)的抽象类：

@[lagom-application](code/ServiceImplementation.scala)

在此示例中，`HelloApplication`使用蛋糕模式混入[[AhcWSComponents|ScalaComponents#Third-party-Components]]，并使用Macwire实现`lazy val lagomService`。这里重要的方法是`lagomServer`方法。Lagom将使用它来发现您的服务绑定并创建一个Play路由器来处理您的服务调用。您会看到我们已经将一个`HelloService`服务描述符绑定到了`HelloServiceImpl`实现中。您绑定的服务描述符的名称将用作服务名称，该名称在跨服务通信中用于标识发出请求的客户端。

我们已经使用Macwire的`wire`宏将依赖项关联到了`HelloServiceImpl`-目前我们的服务实际上没有依赖项，因此我们可以自己手动构建它，但是真正的服务实现不太可能没有依赖项。

该`HelloApplication`是一个抽象类，这样做的原因是，还有一个方法尚未实现，即`serviceLocator`方法。`HelloApplication`扩展了[`LagomApplication`](api/com/lightbend/lagom/scaladsl/server/LagomApplication.html), 其需要`serviceLocator: ServiceLocator`, `lagomServer: LagomServer` 和 `wcClient: WSClient`. 我们提供`wcClient: WSClient`混入`AhcWSComponents`和以编程方式提供 `lagomServer: LagomServer`。典型的应用程序将在不同的环境中使用不同的服务定位器，在开发中，它将使用Lagom开发环境提供的服务定位器，而在生产中，它将使用适合您的生产环境的任何内容，例如由[Akka发现服务定位器](https://github.com/lagom/lagom-akka-discovery-service-locator)。因此，我们的主应用程序蛋糕使该方法抽象，因此可以根据应用程序加载时所处的模式将其正确地混合在一起。

创建了应用程序蛋糕后，我们现在可以编写应用程序加载器。Play加载应用程序的机制是为应用程序提供应用程序加载器。Play会将一些上下文信息传递给此加载器，例如类加载器，运行模式和任何其他配置，以便应用程序可以自我引导。Lagom提供了一种实现此目标的便捷机制[`LagomApplicationLoader`](api/com/lightbend/lagom/scaladsl/server/LagomApplicationLoader.html)：

@[lagom-loader](code/ServiceImplementation.scala)

加载程序有两个必须实现的方法，`load`和`loadDevMode`。您可以看到我们为每种方法混合使用了不同的服务定位器，我们混入[`LagomDevModeComponents`](api/com/lightbend/lagom/scaladsl/devmode/LagomDevModeComponents.html), 它提供开发模式服务定位器, 在开发模式和生产模式中向其注册服务，到目前为止，我们仅提供了[`NoServiceLocator`](api/com/lightbend/lagom/scaladsl/api/ServiceLocator$$NoServiceLocator$.html)作为服务定位 - 这是为每一个查找返回nothing的一个服务定位器。我们将在[[部署到生产|ProductionOverview]]文档中看到如何为生产选择正确的服务定位器。

第三种方法, `describeService`, 是可选的，但工具可以使用它来发现此服务提供的服务API。从这里读取的元数据可以用于配置服务网关和其他组件。

最后，我们需要告诉Play我们应用程序的加载器。我们可以通过添加以下配置来做到`application.conf`这一点:

    play.application.loader = com.example.HelloApplicationLoader


## 定义你自己的组件

要创建一个复杂的应用程序(一个使用持久性，群集，Broker API等的应用程序)，您将需要混入许多组件。一个不错的选择是, 创建混入了这些组件的小型自定义特质, 并通过混入您的小型自定义特质来构建应用程序。这样，您就可以单独测试整个应用程序的各个部分。

假设您的服务使用来自代理主题的消息，并在其中`Orders`得到通知。然后，您的服务将该信息存储到数据库中且最后一步是调用第三方端点进行一些处理。如果您只想测试消息的消费情况和适当的存储，你可以创建`OrderConsumingComponent`特质并混入`LagomServiceClientComponents`和`CassandraPersistenceComponents`, 以便可以消费和存储消息。在您的测试中，您可以用`TestTopicComponents` 扩展 `OrderConsumingComponent`，以提供一个模拟的代理，因此您无需启动代理即可运行测试。最后，在您的应用程序中，您需要混入经过测试的`OrderConsumingComponent` 和 `LagomKafkaClientComponents`。

Lagom提供了几个组件。对于本参考指南，我们按功能将组件分组：

 * [[服务组件|ScalaComponents#Service-Components]]
 * [[持久性和集群组件|ScalaComponents#Persistence-and-Cluster-Components]]
 * [[经纪人API组件|ScalaComponents#Broker-API-Components]]
 * [[服务定位器组件|ScalaComponents#Service-Locator-Components]]
 * [[第三方组件|ScalaComponents#Third-party-Components]]
