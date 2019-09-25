# 使用不可变的对象

不可变对象是创建后无法修改的对象。

不可变对象有两个很大的优点：

* 基于不可变对象的代码更清晰，更可能正确。涉及意外更改的bug根本不会发生。
* 多个线程可以安全地同时访问不可变对象。

在Lagom中，在多个地方都需要不可变的对象，例如:

* 服务请求和响应类型
* 持久实体命令，事件和状态
* 发布和订阅消息

## 可变vs不可变

这是一个可变对象的示例:

@[mutable](code/docs/home/scaladsl/immutable/MutableUser.scala)

setter方法可用于在构造后修改对象。

相比之下，这是一个不可变的对象:

@[immutable](code/docs/home/scaladsl/immutable/ImmutableUser.scala)

所有字段均为final字段，并在构造时分配。没有设置方法。

如您所见，Scala的`case class`编写不可变类非常方便。

请注意，不可变类的包含成员也必须是不可变的。Scala具有不错的不可变集合，您应该在不可变类中使用它。请注意，`scala.collection.Seq`不能保证是不变的。相反，您应该使用`scala.collection.immutable.Seq`或具体的不可变实现，例如`List`或`Vector`。
