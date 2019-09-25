# 服务描述符

Lagom服务由称为服务描述符的接口描述。该接口不仅定义了服务如何调用和实现，还定义了描述该接口如何向下映射到基础传输协议的元数据。通常，服务描述符，其实现和使用情况应该与使用哪种传输方式无关，无论是REST，websocket还是其他某种传输方式。让我们看一个简单的描述符:

@[hello-service](code/ServiceDescriptors.scala)

这个描述符定义了一个只有一个调用的服务，即`sayHello`调用。
`sayHello`是一种返回[`ServiceCall`](api/com/lightbend/lagom/scaladsl/api/ServiceCall.html)类型的方法，这是调用的表示，可以在使用服务时执行该调用，并由服务本身实现。该界面如下所示:

@[service-call](code/ServiceDescriptors.scala)

这里要注意的重要一点是，调用该`sayHello`方法实际上并不执行该调用，而只是获得该调用的句柄，然后可以使用该`invoke`方法来执行该句柄。

ServiceCall接受两个类型参数，Request和Response。该Request参数是传入请求消息的类型，并且所述Response参数是所述输出响应消息的类型。在上面的示例中，它们都是String，因此我们的服务调用仅处理简单的文本消息

[`ServiceCall`](api/com/lightbend/lagom/scaladsl/api/ServiceCall.html) 接受两个类型参数, `Request` 和 `Response`。该 `Request` 参数是传入请求消息的类型, `Response` 参数是输出响应消息的类型。在上面的示例中，它们都是 `String`, 因此我们的服务调用仅处理简单的文本消息。

尽管该`sayHello`方法描述了如何以编程方式调用或实现该调用，但并未描述该调用如何映射到传输上。通过提供该[`descriptor`](api/com/lightbend/lagom/scaladsl/api/Service.html#descriptor:Descriptor)调用的实现来完成此操作，其接口由[`Service`](api/com/lightbend/lagom/scaladsl/api/Service.html)描述。

您可以看到我们正在返回名为`hello`的服务，并且我们正在描述一个调用，即`sayHello`调用。因为这个服务非常简单，所以在这种情况下，我们不需要做任何事情, 只需要将上面示例中定义的服务调用sayHello作为调用方法的方法引用传递给[`call`](api/com/lightbend/lagom/scaladsl/api/Service$.html#call[Request,Response]\(ScalaMethodServiceCall[Request,Response]\)\(MessageSerializer[Request,_],MessageSerializer[Response,_]\):Call[Request,Response])方法即可。

## 调用(Call)标识符

每个服务调用都需要有一个标识符。标识符用于将路由信息提供给客户端和服务的实现，以便可以将有线上的请求映射到适当的后台调用。标识符可以是静态名称或路径，也可以具有动态参数部分，其中动态路径参数从路径中提取出来并传递给服务调用方法。

标识符的最简单类型是名称，默认情况下，该名称设置为与实现它的接口上的方法名相同的名称。在上面的示例中，我们使用了该call方法来创建名称为`sayHello`的服务调用。也可以使用以下[`namedCall`](api/com/lightbend/lagom/scaladsl/api/Service$.html#namedCall[Request,Response]\(String,ScalaMethodServiceCall[Request,Response]\)\(MessageSerializer[Request,_],MessageSerializer[Response,_]\):Call[Request,Response])方法提供自定义名称：

@[call-id-name](code/ServiceDescriptors.scala)

在这种情况下，我们将其命名为`hello`，而不是默认的`sayHello`。使用REST实现时，这意味着此调用的路径为/hello。

### 基于路径的标识符

标识符的第二种类型是基于路径的标识符。这使用URI路径和查询字符串来路由调用，并且可以从中提取动态路径参数。可以使用[`pathCall`](api/com/lightbend/lagom/scaladsl/api/Service$.html#pathCall[Request,Response]\(String,ScalaMethodServiceCall[Request,Response]\)\(MessageSerializer[Request,_],MessageSerializer[Response,_]\):Call[Request,Response])方法配置它们。

通过在路径中声明动态部分，可以从路径中提取动态路径参数。它们以冒号作为前缀，例如，`/order/:id`的路径具有称为`id`的动态部分。Lagom将从路径中提取此参数，并将其传递给服务调用方法。为了将其转换为该方法接受的类型，Lagom将使用隐式提供的[`PathParamSerializer`](api/com/lightbend/lagom/scaladsl/api/deser/PathParamSerializer.html).  Lagom includes many `PathParamSerializer`。Lagom包括许多`PathParamSerializer`开箱即用，比如用于String，Long，Int，Boolean和UUID。这是从路径中提取`long`参数并将其传递给服务调用的示例：

@[call-long-id](code/ServiceDescriptors.scala)

请注意，这次我们为这个方法使用了一个[eta-expanded](https://www.scala-lang.org/files/archive/spec/2.12/06-expressions.html#method-values)引用。这是因为该方法需要一个参数。

当然也可以提取出多个参数，这些参数将按照从URL中提取的顺序传递给您的服务调用方法:

@[call-complex-id](code/ServiceDescriptors.scala)

查询字符串参数也可以从路径中提取，在路径的末尾的一个`?`后面使用一个`&`分隔的列表。例如，以下服务调用使用查询字符串参数来实现分页:

@[call-query-string-parameters](code/ServiceDescriptors.scala)

当您使用 `call`, `namedCall` 或 `pathCall`, 如果Lagom把它映射到REST, Lagom将作出最大努力用语义的方式将它映射为REST. 例如, 如果有请求消息，它将使用 `POST` 方法，而如果没有请求消息，则将使用 `GET`。

### REST标识符

最后一种标识符类型是REST标识符。REST标识符旨在在创建语义REST API时使用。它们既使用路径(与基于路径的标识符一样)，也使用请求方法来标识它们。可以使用以下[`restCall`](api/com/lightbend/lagom/scaladsl/api/Service$.html#restCall[Request,Response]\(Method,String,ScalaMethodServiceCall[Request,Response]\)\(MessageSerializer[Request,_],MessageSerializer[Response,_]\):Call[Request,Response])方法配置它们:

@[call-rest](code/ServiceDescriptors.scala)

## 消息

Lagom中的每个服务调用都有一个请求消息类型和一个响应消息类型。如果不使用请求或响应消息，则可以在其位置使用`akka.NotUsed`。请求和响应消息类型分为两类：严格和流式。

### 严格的消息

严格消息是可以由简单Scala对象(通常是用例类)表示的单个消息。该消息将被缓冲到内存中，然后解析, 比如转成JSON。当两种消息类型都严格时，该调用被称为同步调用，即发送和接收请求，然后发送和接收响应。调用者和被调用者已同步他们的通信。

到目前为止，我们已经看到的所有服务调用示例都使用了严格的消息，例如，上面的订单服务描述符接受并返回条目和订单。输入值直接传递到服务调用，并直接从服务调用返回，并且这些值在发送之前被序列化到内存中的JSON缓冲区，并在从JSON反序列化之前完全读入内存。

### 流消息

流消息是[`Source`](https://doc.akka.io/api/akka/2.6/akka/stream/scaladsl/Source.html)类型的消息。`Source`是一个[Akka streams](https://doc.akka.io/docs/akka/2.6/stream/?language=scala) API，允许异步流和消息处理。这是流服务调用的示例:

@[call-stream](code/ServiceDescriptors.scala)

该服务调用具有严格的请求类型和流式响应类型。此实现可能会返回一个`Source`，它在指定的时间间隔发送输入tick消息`String`。

双向流式调用可能如下所示:

@[hello-stream](code/ServiceDescriptors.scala)

在这种情况下，服务器可能会返回一个`Source`，将请求流中接收到的每条消息转换为前缀为`Hello`的消息。

Lagom将为流选择合适的传输方式，通常是WebSockets。WebSocket协议支持双向流，因此是流的一个很好的通用选项。当请求或响应消息只有一个是流时，Lagom将通过发送或接收单个消息来实现发送和接收严格消息，然后将WebSocket保持打开状态，直到另一个方向关闭为止。否则，当任一方向关闭时，Lagom将关闭WebSocket。

### 消息序列化

使用类型类提供了用于请求和响应的消息序列化器。`call`, `namedCall`, `pathCall` 和 `restCall`方法中的每一个都获得隐式[`MessageSerializer`](api/com/lightbend/lagom/scaladsl/api/deser/MessageSerializer.html)为每个请求和响应消息。Lagom提供了开箱即用的String消息序列化程序，以及将Play JSON [`Format`](https://www.playframework.com/documentation/2.6.x/api/scala/play/api/libs/json/Format.html)类型类隐式转换为消息序列化程序的序列化程序。

#### 使用 Play JSON

Play JSON提供了一个基于函数类型的库，用于组合JSON格式化程序。有关如何使用此库的详细文档，请参阅[Play文档](https://www.playframework.com/documentation/2.6.x/ScalaJsonCombinators)。现在，我们只看一下如何使用Play的JSON格式宏为用例类定义JSON格式。

假设您有一个如下的 `User` 用例类：

@[user-class](code/ServiceDescriptors.scala)

可以在 `User` 伴生对象上定义Play JSON格式，如下所示：

@[user-format](code/ServiceDescriptors.scala)

此格式将以以下格式生成和解析JSON：

```json
{
  "id": 12345,
  "name": "John Smith",
  "email": "john.smith@example.org"
}
```

可以通过将字段设置为`Option`类型来使字段成为可选字段，这将意味着，如果不存在属性，将不会解析JSON格式失败，并且在生成JSON时，它将不会生成该属性。

通过定义 `User` 伴随对象上的格式，由于Scala的隐式作用域规则，我们可以确保该格式将在需要时自动使用。这意味着，除了声明格式之外，无需执行其他任何工作来确保将这种格式用于`MessageSerializer`。

请注意，如果您的用例类引用了另一个非原始类型，例如另一个用例类，则还需要为该用例类定义一种格式。

#### 编写自定义消息序列化器

您还可以编写自定义消息序列化程序，例如，以使用协议缓冲区或其他消息格式类型。有关更多信息，请参见[[消息序列化器文档|MessageSerializers]]。
