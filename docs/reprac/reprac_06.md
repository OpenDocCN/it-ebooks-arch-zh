# 后台与服务篇

### PHP？最好的语言

### RESTful 与服务化

### 设计 RESTful API

> REST 从资源的角度来观察整个网络，分布在各处的资源由 URI 确定，而客户端的应用通过 URI 来获取资源的表征。获得这些表征致使这些应用程序转变了其状态。随着不断获取资源的表征，客户端应用不断地在转变着其状态，所谓表征状态转移。

因为我们需要的是一个 Machine 到 Machine 沟通的平台，需要设计一个 API。而设计一个 API 来说，RESTful 是很不错的一种选择，也是主流的选择。而设计一个 RESTful 服务，的首要步骤便是设计资源模型。

### 资源

互联网上的一切信息都可以看作是一种资源。

| HTTP Method | Operation Performed |
| --- | --- |
| GET | Get a resource (Read a resource) |
| POST | Create a resource |
| PUT | Update a resource |
| DELETE | Delete Resource |

设计 RESTful API 是一个有意思的话题。下面是一些常用的 RESTful 设计原则:

*   组件间交互的可伸缩性
*   接口的通用性
*   组件的独立部署
*   通过中间组件来减少延迟、实施安全策略和封装已有系统

判断是否是 RESTful 的约束条件

*   客户端-服务器分离
*   无状态
*   可缓存
*   多层系统
*   统一接口
*   随需代码（可选）

### 微服务

### 微内核

这只是由微服务与传统架构之间对比而引发的一个思考，让我引一些资料来当参考吧.

> 单内核：也称为宏内核。将内核从整体上作为一个大过程实现，并同时运行在一个单独的地址空间。所有的内核服务都在一个地址空间运行，相互之间直接调用函数，简单高效。微内核：功能被划分成独立的过程，过程间通过 IPC 进行通信。模块化程度高，一个服务失效不会影响另外一个服务。Linux 是一个单内核结构，同时又吸收了微内核的优点：模块化设计，支持动态装载内核模块。Linux 还避免了微内核设计上的缺陷，让一切都运行在内核态，直接调用函数，无需消息传递。

对就的微内核便是:

> 微内核――在微内核中，大部分内核都作为单独的进程在特权状态下运行，他们通过消息传递进行通讯。在典型情况下，每个概念模块都有一个进程。因此，假如在设计中有一个系统调用模块，那么就必然有一个相应的进程来接收系统调用，并和能够执行系统调用的其他进程（或模块）通讯以完成所需任务。

如果读过《操作系统原理》及其相关书籍的人应该很了解这些，对就的我们就可以一目了然地解决我们当前是的微服务的问题。

### 微服务

文章的来源是 James Lewis 与 Martin Fowler 写的[Microservices](http://martinfowler.com/articles/microservices.html)。对就于上面的

*   monolithic kernel
*   microkernel

与文中的

*   monolithic services
*   microservices

我们还是将其翻译成`微服务`与`宏服务`。

引起原文中对于微服务的解释:

> 简短地说，微服务架构风格是一种使用一套小服务来开发单个应用的方式途径，每个服务运行在自己的进程中，通过轻量的通讯机制联系，经常是基于 HTTP 资源 API，这些服务基于业务能力构建，能够通过自动化部署方式独立部署，这些服务自己有一些小型集中化管理，可以是使用不同的编程语言编写，正如不同的数据存储技术一样。

原文是:

> In short, the microservice architectural style [1](http://repractise.phodal.com/img/a-arch/blog-mobile.jpg) is an approach to developing a single application as a suite of small services, each running in its own process and communicating with lightweight mechanisms, often an HTTP resource API. These services are built around business capabilities and independently deployable by fully automated deployment machinery. There is a bare mininum of centralized management of these services, which may be written in different programming languages and use different data storage technologies.

而关于微服务的提出是早在 2011 年的 5 月份

> The term “microservice” was discussed at a workshop of software architects near Venice in May, 2011 to describe what the participants saw as a common architectural style that many of them had been recently exploring.

### 微服务思考

简单地与微内核作一些对比。微内核，**微内核部分经常只但是是个消息转发站**，而微服务从某种意义上也是如此，他们都有着下面的优点。

*   有助于实现模块间的隔离
*   在不影响系统其他部分的情况下，用更高效的实现代替现有文档系统模块的工作将会更加容易。

对于微服务来说

*   每个服务本身都是很简单的
*   对于每个服务，我们可以选择最好和最合适的工具来开发
*   系统本质上是松耦合的
*   不同的团队可以工作在不同的服务中
*   可以持续发布，而其他部分还是稳定的

从某种意义上来说微服务更适合于大型企业架构，而不是一般的应用，对于一般的应用来说他们的都在同一台主机上。无力于支付更多的系统开销，于是如**微服务不是免费的午餐**一文所说

*   微服务带来很多的开销操作
*   大量的 DevOps 技能要求
*   隐式接口
*   重复努力
*   分布式系统的复杂性
*   异步性是困难的！
*   可测试性挑战

因而不得不再后面补充一些所知的额外的东西。

### 微服务与持续集成

针对于同样的话题，开始了解其中的一些问题。当敏捷的思想贯穿于开发过程时，我们不得不面对持续集成与发布这样的问题。我们确实可以在不同的服务下工作，然而当我们需要修改 API 时，就对我们的集成带来很多的问题。我们需要同时修改两个 API！我们也需要同时部署他们！

#### 微服务与测试

相比较的来说，这也是另外的一个挑战。测试对于项目开发来说是不可缺少的，而当我们的服务一个个隔离的时候，我们的测试不得不去 mock 一个又一个的服务。在有些时候修复这些测试可能比添加这个功能花费的时间还多。

不过他更适合那些喜欢不同技术栈的程序员。

### 参考

[Microservices - Not A Free Lunch!](http://highscalability.com/blog/2014/4/8/microservices-not-a-free-lunch.html)

[Microservices](http://martinfowler.com/articles/microservices.html)