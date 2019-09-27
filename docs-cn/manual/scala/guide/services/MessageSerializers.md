# 消息序列化器

开箱即用，Lagom使用Play JSON序列化请求和响应消息。您还可以使用自己喜欢的任何有线协议（从JSON到protobuf再到XML）定义用于您的类型的自定义序列化程序。

## Lagom如何选择一个消息序列化器

当你定义你的服务描述符时,  `call`, `namedCall`, `pathCall`, `restCall`和`topic`方法都接受['隐式'](https://docs.scala-lang.org/zh-cn/tour/implicit-parameters.html)[`MessageSerializer`](api/com/lightbend/lagom/scaladsl/api/deser/MessageSerializer.html)参数来处理您的服务调用时使用的消息。与Scala中的所有隐式参数一样，您可以让Scala编译器为您隐式解析这些参数，也可以显式传递它们。

例如，这显示了如何显式传递默认的Lagom `String`序列化器：

@[explicit-serializers](code/MessageSerializers.scala)

我们在[[服务描述符|ServiceDescriptors#Using-Play-JSON]]文档中看到了如何通过在样例类伴随对象上声明隐式的Play JSON `Format`，Lagom会将其用于该类型的消息。之所以起作用，是因为Lagom提供了一个隐式的`MessageSerializer`, 其包含Play JSON `Format`。这是`MessageSerializer`伴生对象上的`jsValueFormatMessageSerializer`方法。

`MessageSerializer`伴生对象还为你可能想用的其他常用的, 非JSON的有效载荷提供隐式。例如，任何时候您的请求或响应类型为`NotUsed`, `Done` 或 `String`, 这些的默认序列化器将被使用。Lagom还附带了对`ByteString`序列化器(aka `noop`)的支持，因此可以直接访问wire-level有效负载。

JSON消息序列化器格式化也可以显式使用。假设您有一条带有id属性的消息，对于一个服务调用，您希望使用Play JSON宏为您提供的默认格式，但是在另一个中，您想要一种不同的格式，一种id字段在JSON中被命名为`identifier`的格式。您可以提供两种不同的格式:

@[case-class-two-formats](code/MessageSerializers.scala)

您可以看到我们已经将其中一个设成隐式，因此，如果我们用隐式方案来执行它的工作，它将被选中。然后，这个非隐式可以在服务调用描述符中被显式的传递:

@[descriptor-two-formats](code/MessageSerializers.scala)

## 自定义序列化器

JSON可能不是您想要使用的唯一一种连接格式类型。Lagoms [`MessageSerializer`](api/com/lightbend/lagom/scaladsl/api/deser/MessageSerializer.html)特质可以用于实现自定义序列化器。

正如我们[[已经看到的|ServiceDescriptors#Messages]]，Lagom中有两种消息，严格消息和流消息。对于这两种类型的消息，Lagom提供了两个`MessageSerializer`子接口, [`StrictMessageSerializer`](api/com/lightbend/lagom/scaladsl/api/deser/StrictMessageSerializer.html) 和 [`StreamedMessageSerializer`](api/com/lightbend/lagom/scaladsl/api/deser/StreamedMessageSerializer.html)，其中主要区别在于连接(wire)格式，它们序列化成什么格式和从什么格式反序列化。严格的消息序列化器要序列化成`ByteString`和从`ByteString`反序列化，也就是说，它们严格地在内存中工作，而流消息序列化器则与流一起工作，也就是，`Source[ByteString, _]`。

在研究如何实现序列化器之前，需要先介绍一些基本概念。

### 消息协议

Lagom有一个消息协议的概念。消息协议使用[`MessageProtocol`](api/com/lightbend/lagom/scaladsl/api/transport/MessageProtocol.html)类型表示，并且具有三个属性，即内容类型，字符集和版本。所有这些属性都是可选的，一个消息序列化程序可能会用到也可能不会用到。

消息协议大致转换为HTTP `Content-Type`和`Accept`头，如果使用了对版本进行编码的mime类型方案，则可能从这些协议中提取版本，也可能从URL中提取版本, 这取决于服务的配置方式。

### 内容协商

Lagom消息序列化器能够使用内容协商来决定使用什么协议来彼此通信。这可以用来指定不同的传输格式，比如JSON和XML，以及不同的版本。

Lagom的内容协商反映了与HTTP相同的功能。对于请求消息，客户端将选择其要使用的任何协议，因此在那里无需进行协商。然后，服务器使用客户端发送的消息协议来决定如何反序列化请求。

对于响应，客户端发送可接收的消息协议列表，服务器要从该列表中选择一个协议进行响应。然后，客户端将读取服务器选择的协议，并使用该协议反序列化响应。

### 协商的序列化器

作为内容协商的结果，Lagom `MessageSerializer`不会直接对消息进行序列化和反序列化，而是提供了用于协商消息协议的方法, 其返回[`NegotiatedSerializer`](api/com/lightbend/lagom/scaladsl/api/deser/MessageSerializer$$NegotiatedSerializer.html) 或 [`NegotiatedDeserializer`](api/com/lightbend/lagom/scaladsl/api/deser/MessageSerializer$$NegotiatedDeserializer.html)。这些协商的类实际上负责进行序列化和反序列化。

让我们看一个内容协商的例子。假设我们要实现一个自定义String `MessageSerializer`，可以序列化为纯文本或JSON, 实际根据客户端的请求。如果您有一些客户端以JSON形式发送文本主体，而另一些客户端以纯文本形式发送文本主体，那么这可能会很有用，也许其中一个客户端是旧客户端，它以一种方式执行操作，但是现在您希望使用新客户端用另一种方式来执行。

首先，我们将为纯文本字符串实现`NegotiatedSerializer`：

@[plain-text-serializer](code/MessageSerializers.scala)

该`protocol`方法返回这个序列化器序列化到什么格式的协议，您可以看到我们正在传递序列化器将在构造函数中使用的`charset`。该`serialize`方法是从`String`直接转换到`ByteString`。

接下来，我们将实现相同的东西，但是要序列化为JSON：

@[json-text-serializer](code/MessageSerializers.scala)

在这里，我们使用Play JSON将`String`转换为JSON字符串。

现在让我们实现纯文本反序列化器：

@[plain-text-deserializer](code/MessageSerializers.scala)

同样，我们正在将`charset`作为一个构造函数的参数，我们简单的将`ByteString`转换为`String`。

类似的，我们有一个JSON文本反序列化器：

@[json-text-deserializer](code/MessageSerializers.scala)

现在我们已经实现了协商的序列化器和反序列化器，是时候实现`MessageSerializer`进行实际协议协商了。我们的类将扩展`StrictMessageSerializer`：

@[text-serializer](code/MessageSerializers.scala)

我们需要做的下一件事是定义我们接受的协议。客户端将使用它来设置`Accept`标头：

@[text-serializer-protocols](code/MessageSerializers.scala)

您可以看到此序列化器支持文本和json协议。需要注意的一件事是，我们没有在文本协议中设置字符集，这是因为我们不需要具体说明它，我们可以接受服务器选择的任何字符集。

现在让我们实现该`serializerForRequest`方法。客户端使用它来确定要用于请求的序列化器。因为在此阶段，服务器与客户端之间没有任何通信发生，所以无法进行协商，因此客户端只需选择默认的序列化器，在这种情况下，就是`utf-8`纯文本序列化器：

@[text-serializer-request](code/MessageSerializers.scala)

接下来，我们将实现该`deserializer`方法。它既可以用于服务器为请求选择反序列化器，也可以用于客户端为响应选择反序列化器。传入的`MessageProtocol`是随请求或响应发送的内容类型，我们需要检查它是否是可以反序列化的内容类型，然后返回适当的内容类型：

@[text-deserializer](code/MessageSerializers.scala)

请注意，如果未指定内容类型，我们将返回默认的反序列化器。我们也可以抛出一个异常用于在此处失败，但是最好不要这样做，因为某些底层传输不允许传递带有消息的内容类型。例如，如果此请求用于WebSocket请求，Web浏览器不允许您设置WebSocket请求的内容类型。如果未设置任何内容类型，则返回默认值，以确保最大的可移植性。

接下来，我们将实现`serializerForResponse`方法。这将获得客户端发送的可接受协议的列表，并选择一个用于序列化响应。如果找不到一个支持的，则会引发异常。请注意，如果任何属性的值为空，则表示客户端愿意接受任何内容，如果客户端未指定任何接受协议，则同样如此。

@[text-serializer-response](code/MessageSerializers.scala)

## 例子

### 协议缓冲区序列化器

[协议缓冲区](https://developers.google.com/protocol-buffers/)是JSON的高性能语言中立替代品，对于服务之间的内部通信而言，这是一个特别好的选择。这个示例说明如何为`protoc`生成的`Order`类编写一个`MessageSerializer`：

@[protobuf](code/MessageSerializers.scala)

请注意，这个`MessageSerializer`不会尝试进行任何内容协商。在许多情况下，内容协商是过度的，如果您不需要它，则不必实现它。
