# 附加路由器

从Lagom 1.5.0开始，可以使用附加的Play路由器扩展Lagom服务。

当将Lagom与现有的Play路由器(比如[Play gRPC路由器](https://developer.lightbend.com/docs/play-grpc/0.6.0/lagom/serving-grpc.html?language=scala)或您可以使用的任何其他Play路由器)集成时，这特别有用。

在连接Lagom服务器时，您将添加一个附加路由器。在连接Lagom服务器后，将其他Play路由器追加到该服务器。

@[lagom-application-some-play-router](code/AdditionalRouters.scala)

## 文件上传示例

以下示例显示了如何将文件上传端点添加到现有的Lagom服务。

该示例基于[ScalaSirdRouter](https://www.playframework.com/documentation/2.7.x/ScalaSirdRouter)，可让您以编程方式构建Play路由器。它添加了一个额外的路径(`/api/files`)，用于接收多部分表单数据的POST调用。

@[file-upload-router](code/AdditionalRouters.scala)

在应用程序加载器中，您可以连接路由器并将其追加到Lagom服务器。

@[lagom-application-file-upload](code/AdditionalRouters.scala)

这个`/api/files`路径现在可在您的Lagom服务上使用了：

```bash
curl -X POST -F "data=@somefile.txt" -v  http://localhost:65499/api/files
```

> 请注意，在该示例中，我们没有使用服务网关来访问应用程序。我们使用服务端口(在本例中为65499)直接调用它。

## 服务网关注意事项

附加路由器不属于您的应用程序的`ServiceDescriptor`，因此无法在开发模式下自动发布为[[服务网关|ServiceLocator#Service-Gateway]]的端点。

如果你想要通过网关访问附加路由器，则需要在`ServiceDescriptor`定义中为其明确添加ACL(访问控制列表)。

@[hello-service](code/AdditionalRouters.scala)

一旦路径在服务网关上发布后，您就可以调用了：

```bash
curl -X POST -F "data=@somefile.txt" -v  http://localhost:9000/api/files
```

> 注意端口9000(Lagom的开发模式服务网关)的用途

## Lagom客户端的注意事项

附加路由器不是服务API的一部分，因此无法从生成的Lagom客户端访问。Lagom客户端只能访问在服务特质上定义的方法。

附加路由器只是公开的HTTP端点的一部分。要进行访问，您将需要使用HTTP客户端，例如：[Play-WS](https://www.playframework.com/documentation/2.7.x/ScalaWS)
