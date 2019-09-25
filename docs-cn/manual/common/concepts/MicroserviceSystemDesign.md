# 设计您的微服务系统

Bonér的[反应式微服务架构：分布式系统的设计原理](https://info.lightbend.com/COLL-20XX-Reactive-Microservices-Architecture-RES-LP.html)是微服务系统架构师的基础阅读。要在这个复杂的领域变得熟练，需要时间和经验。Lagom通过提供自以为是(opinionated)的API，实现的支持功能以及适当的默认值来提供实用指南。

我们建议从小开始。首先，确定对可以使用异步消息的简单微服务的需求。它不必复杂，甚至不需要提供很多价值。简单性降低了与部署相关的风险，并且可以快速赢得胜利。接下来，在体系结构级别，提取一个可以划分的核心服务。将其划分为微服务系统。当您一次解决一个问题时，您和您的团队将不断学习，并将变得越来越有效。使用诸如[领域驱动设计](https://en.wikipedia.org/wiki/Domain-driven_design) (DDD)之类的方法可以帮助您的组织应对企业系统固有的复杂性。

## 更换整体

在设计微服务系统以替换现有的整体组件时，您的方法可能会有所不同，具体取决于需求和现有架构。例如，如果整体设计合理，则您可以首先将旧组件分离，然后一次集中精力迁移功能。如果无法接受合理的停机时间，则需要详细的计划，以将功能从旧系统切换到新系统。

例如，想象一个处理许多部门核心业务功能的企业整体应用程序。它可能具有会计，销售，市场营销和运营所使用的库存功能 --- 每个模块都以不同的方式使用它。DDD鼓励将如此庞大而笨拙的模型分解为[有界上下文](https://martinfowler.com/bliki/BoundedContext.html)。每个有界上下文都定义了一个适用于特定团队的边界，解决了特定的用途，并包括为该上下文具体化系统所必需的数据模式和物理元素。使用有界上下文可以使小型团队一次专注于一个上下文并并行工作。

在第一个实施阶段，您可以修改整体，以开始发布特定用例的会计部门感兴趣的库存事件。下图说明了如何将其发布到[Kafka](https://kafka.apache.org/intro)，Kafka是一个支持发布/订阅（pub/sub）并作为性能和可靠性的群集运行的流平台。

[[MonolithPhaseOne.png]]

接下来，如下图所示，您可以创建一个微服务，该微服务订阅该主题并处理数据。微服务的多个实例提供了可伸缩性和容错能力。在测试和微调微服务时，旧功能仍然存在。

[[MonolithPhaseTwo.png]]

接下来，如图所示，您可以修改整体以对新的微服务进行远程HTTP调用，而不是调用其自己的内部业务逻辑。

[[MonolithPhaseThree.png]]

当您确信新的微服务运行良好时，可以从整体中删除内部业务逻辑，然后转到下一个微服务或一组微服务。这个高级示例只是您可能选择从整体中移动功能的众多方式之一。

<!---The following diagram illustrates a first try at decomposing a monolith. **A** represents a simple piece of functionality that was redesigned as a microservice. With the microservice instances online, code in the monolith that uses that functionality now locates available instances through the Service Gateway and communicates with them using the Kafka message broker. -->
