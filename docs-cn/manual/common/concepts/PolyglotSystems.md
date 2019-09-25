# 使用Lagom的多语言系统

Lagom并不希望您系统中的每个服务都将是Lagom微服务。毕竟，首先使用微服务的一大优势是为每种服务务实地选择最佳的语言和技术。Lagom对标准通信协议的惯用用法以及与设计驱动的API方法的大体兼容，使其可以在这种多语言系统中很好地工作。

Lagom服务调用映射到用于同步通信的标准HTTP和用于流和异步消息传递的WebSockets。任何支持这些协议的语言或框架都可以轻松地使用Lagom服务。反过来，Lagom服务可以轻松地与任何公开REST API的服务通信。

Lagom服务的API指定该服务如何使用HTTP。REST服务调用由HTTP方法和URI标识。可以自定义请求和响应头。Lagom消息已被序列化, 默认情况下，为普通JSON，使用惯用映射库，使JSON在网络上的表示方式透明。

<!--- Lagom uses the same interfaces in Lagom to Lagom service communication. --->
