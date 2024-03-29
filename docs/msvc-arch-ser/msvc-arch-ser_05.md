# 微服务实战（四）：服务发现的可行方案以及实践案例

> [`dockone.io/article/771`](http://dockone.io/article/771)

这是关于使用微服务架构创建应用系列的第四篇文章。第一篇介绍了微服务架构的模式，讨论了使用微服务架构的优缺点。第二和第三篇描述了微服务架构内部的通讯机制。这篇文章中，我们将会探讨服务发现相关问题。

## 为什么要使用服务发现?

设想一下，我们正在写代码使用了提供 REST API 或者 Thrift API 的服务，为了完成一次服务请求，代码需要知道服务实例的网络位置（IP 地址和端口）。传统应用都运行在物理硬件上，服务实例的网络位置都是相对固定的。例如，代码可以从一个经常变更的配置文件中读取网络位置。

而对于一个现代的，基于云微服务的应用来说，这却是一个很麻烦的问题。其架构如图所示：
![1.png](img/6bb645dc0ddf13b49a2c58e9a94393d1.jpg)
服务实例的网络位置都是动态分配的，而且因为扩展、失效和升级等需求，服务实例会经常动态改变，因此，客户端代码需要使用一种更加复杂的服务发现机制。

目前有两大类服务发现模式：[客户端发现](http://microservices.io/patterns/client-side-discovery.html)和[服务端发现](http://microservices.io/patterns/server-side-discovery.html)。

我们先来来讨论一下客户端发现。

## 客户端发现模式

当使用[客户端发现模式](http://microservices.io/patterns/client-side-discovery.html)时，客户端负责决定相应服务实例的网络位置，并且对请求实现负载均衡。客户端从一个服务注册服务中查询，其中是所有可用服务实例的库。客户端使用负载均衡算法从多个服务实例中选择出一个，然后发出请求。

下图显示的是这种模式的架构图：
![2.png](img/710d37e44967ba805cadc33d6737c1cf.jpg)
服务实例的网络位置是在启动时注册到服务注册表中，并且在服务终止时从注册表中删除。服务实例注册信息一般是使用心跳机制来定期刷新的。

[Netflix OSS](https://netflix.github.io/)提供了一种非常棒的客户端发现模式。[Netflix Eureka](https://github.com/Netflix/eureka)是一个服务注册表，为服务实例注册管理和查询可用实例提供了 REST API 接口。[Netflix Ribbon](https://github.com/Netflix/ribbon)是一种 IPC 客户端，与 Eureka 合同工作实现对请求的负载均衡。我们会在后面详细讨论 Eureka。

客户端发现模式也是优缺点分明。这种模式相对比较直接，而且除了服务注册表，没有其它改变的因素。除此之外，因为客户端知道可用服务注册表信息，因此客户端可以通过使用哈希一致性（hashing consistently）变得更加聪明，更加有效的负载均衡。

而这种模式一个**最大的缺点**是需要针对不同的编程语言注册不同的服务，在客户端需要为每种语言开发不同的服务发现逻辑。

我们分析过客户端发现后，再看看服务端发现。

## 服务端发现模式

另外一种服务发现的模式是服务端发现模式（server-side discovery pattern），下图展现了这种模式的架构图：
![3.png](img/5861d94f63ff8af0fdb0af5f0f187e9f.jpg)
客户端通过负载均衡器向某个服务提出请求，负载均衡器向服务注册表发出请求，将每个请求转发往可用的服务实例。跟客户端发现一样，服务实例在服务注册表中注册或者注销。

AWS Elastic Load Balancer（ELB）是一种服务端发现路由的例子，ELB 一般用于均衡从网络来的访问流量，也可以使用 ELB 来均衡 VPC 内部的流量。客户端使用 DNS，通过 ELB 发出请求（HTTP 或者 TCP）。ELB 负载均衡器负责在注册的 EC2 实例或者 ECS 容器之间均衡负载，并不存在一个分离的服务注册表，而 EC2 实例和 ECS 实例也向 ELB 注册。

HTTP 服务和类似 NGINX 和[NGINX Plus](https://www.nginx.com/products/)的负载均衡器都可以作为服务端发现均衡器。例如，[这篇博文](https://www.airpair.com/scalable-architecture-with-docker-consul-and-nginx)就描述如何使用[Consul Template](https://github.com/hashicorp/consul-template)来动态配置 NGINX 反向代理。Consul Template 是周期性从存放在[Consul Template 注册表](https://www.consul.io/)中配置数据重建配置文件的工具。当文件发生变化时，会运行一个命令。在如上博客中，Consul Template 产生了一个 nginx.conf 文件，用于配置反向代理，然后运行一个命令，告诉 NGINX 重新调入配置文件。更复杂的例子可以用 HTTP API 或者 DNS 动态重新配置 NGINX Plus。

某些部署环境，例如[Kubernetes](https://github.com/kubernetes/kubernetes/blob/master/docs/design/architecture.md)和[Marathon](https://mesosphere.github.io/marathon/docs/service-discovery-load-balancing.html)在集群每个节点上运行一个代理，此代理作为服务端发现负载均衡器。为了向服务发出请求，客户端使用主机 IP 地址和分配的端口通过代理将请求路由出去。代理将次请求透明的转发到集群中可用的服务实例。

服务端发现模式也有优缺点。最大的优点是客户端无需关注发现的细节，客户端只需要简单的向负载均衡器发送请求，实际上减少了编程语言框架需要完成的发现逻辑。而且，如上说所，某些部署环境免费提供以上功能。

这种模式也有缺陷，除非部署环境提供负载均衡器，否则负载均衡器是另外一个需要配置管理的高可用系统功能。

## 服务注册表

[服务注册表](http://microservices.io/patterns/service-registry.html)是服务发现很重要的部分，它是包含服务实例网络地址的数据库。服务注册表需要高可用而且随时更新。客户端可以缓存从服务注册表获得的网络地址。然而，这些信息最终会变得过时，客户端也无法发现服务实例。因此，服务注册表由若干使用复制协议保持同步的服务器构成。

如前所述，[Netflix Eureka](https://github.com/Netflix/eureka)是一个服务注册表很好地例子，提供了 REST API 注册和请求服务实例。 服务实例使用 POST 请求注册网络地址，每 30 秒必须使用 PUT 方法更新注册表，使用 HTTP DELETE 请求或者实例超时来注销。可以想见，客户端可以使用 HTTP GET 请求接受注册服务实例信息。

Netflix 通过在每个 AWS EC2 域运行一个或者多个 Eureka 服务[实现高可用性](https://github.com/Netflix/eureka/wiki/Configuring-Eureka-in-AWS-Cloud)，每个 Eureka 服务器都运行在拥有[弹性 IP 地址](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html)的 EC2 实例上。DNS TEXT 记录用于存储 Eureka 集群配置，其中存放从可用域到一系列 Eureka 服务器网络地址的列表。当 Eureka 服务启动时，向 DNS 请求接受 Eureka 集群配置，确认同伴位置，给自己分配一个未被使用的弹性 IP 地址。

Eureka 客户端—服务和服务客户端—向 DNS 请求发现 Eureka 服务的网络地址，客户端首选使用同一域内的服务。然而，如果没有可用服务，客户端会使用另外一个可用域的 Eureka 服务。

另外一些服务注册表例子包括：

*   [etcd](https://github.com/coreos/etcd) – 是一个高可用，分布式的，一致性的，键值表，用于共享配置和服务发现。两个著名案例包括 Kubernetes 和 Cloud Foundry。
*   [consul](https://www.consul.io/) – 是一个用于发现和配置的服务。提供了一个 API 允许客户端注册和发现服务。Consul 可以用于健康检查来判断服务可用性。
*   [Apache ZooKeeper](http://zookeeper.apache.org/) – 是一个广泛使用，为分布式应用提供高性能整合的服务。Apache ZooKeeper 最初是 Hadoop 的子项目，现在已经变成顶级项目。

另外，前面强调过，某些系统，例如 Kubernetes、Marathon 和 AWS 并没有独立的服务注册表，对他们来说，服务注册表只是一个内置的功能。

现在我们来看看服务注册表的概念，看看服务实例是如何在注册表中注册的。

## 服务注册选项

如前所述，服务实例必须向注册表中注册和注销，如何注册和注销也有一些不同的方式。一种方式是服务实例自己注册，也叫自注册模式（self-registration pattern）；另外一种方式是为其它系统提供服务实例管理的，也叫第三方注册模式（third party registration pattern）。我们来看看自注册模式。

### 自注册方式

当使用自注册模式时，服务实例负责在服务注册表中注册和注销。另外，如果需要的话，一个服务实例也要发送心跳来保证注册信息不会过时。下图描述了这种架构：
![4.png](img/3a8f7a14be54030b3eff0661bb564ca2.jpg)
一个很好地例子是 Netflix OSS Eureka client。Eureka 客户端负责处理服务实例的注册和注销。Spring Cloud project，实现了多种模式，包括服务发现，使得向 Eureka 服务实例自动注册时更容易。可以用@EnableEurekaClient 注释 Java 配置类。

自注册模式也有优缺点。一个优点是，相对简单，不需要其他系统功能。而一个主要缺点则是，把服务实例跟服务注册表联系起来。必须在每种编程语言和框架内部实现注册代码。

另外一个方法，不需要连接服务和注册表，则是第三方注册模式。

## 第三方注册模式

当使用[第三方注册模式](http://microservices.io/patterns/3rd-party-registration.html)时，服务实例并不负责向服务注册表注册，而是由另外一个系统模块，叫做服务管理器，负责注册。服务管理器通过查询部署环境或订阅事件来跟踪运行服务的改变。当管理器发现一个新可用服务，会向注册表注册此服务。服务管理器也负责注销终止的服务实例。下图是这种模式的架构图。
![5.png](img/b7b77c86ecf21983575fb6c29026ea02.jpg)
一个服务管理器的例子是开源项目[Registrator](https://github.com/gliderlabs/registrator)，负责自动注册和注销被部署为 Docker 容器的服务实例。Reistrator 支持多种服务管理器，包括 etcd 和 Consul。

另外一个服务管理器例子是[NetflixOSS Prana](https://github.com/netflix/Prana)，主要面向非 JVM 语言开发的服务，也称为附带应用（sidecar application），Prana 使用 Netflix Eureka 注册和注销服务实例。

服务管理器是部署环境内置的模块。有自动扩充组创建的 EC2 实例可以自向 ELB 自动注册，Kubernetes 服务自动注册并且对发现服务可用。

第三方注册模式也是优缺点都有。主要的优点是服务跟服务注册表是分离的，不需要为每种编程语言和架构完成服务注册逻辑，替代的，服务实例是通过一个集中化管理的服务进行管理的。

一个缺点是，除非这种服务被内置于部署环境中，否则也需要配置管理一个高可用的系统。

## 总结

在一个微服务应用中，服务实例运行环境是动态变化的。实例网络地址也是动态变化的，因此，客户端为了访问服务必须使用服务发现机制。

服务发现关键部分是[服务注册表](http://microservices.io/patterns/service-registry.html)，也就是可用服务实例的数据库。服务注册表提供一种注册管理 API 和请求 API。服务实例使用注册管理 API 来实现注册和注销。

请求 API 用于发现可用服务实例，相对应的，有两种主要服务发现模式：[客户端发现](http://microservices.io/patterns/client-side-discovery.html)和[服务端发现](http://microservices.io/patterns/server-side-discovery.html)。

在使用客户端发现的系统中，客户端向服务注册表发起请求，选择可用实例，然后发出服务请求

而在使用服务端发现的系统中，客户端通过路由转发请求，路由器向服务注册表发出请求，转发此请求到某个可用实例。

服务实例注册和注销主要有两类方式。一种是服务实例自动注册到服务注册表中，也就是自注册模式；另外一种则是某个系统模块负责处理注册和注销，也就是第三方注册模式。

在某些部署环境中，需要配置自己的服务发现架构，例如：[Netflix Eureka](https://github.com/Netflix/eureka)、[etcd](https://github.com/coreos/etcd)或者[Apache ZooKeeper](http://zookeeper.apache.org/)。而在另外一些部署环境中，则自带了这种功能，例如 Kubernetes 和 Marathon 负责处理服务实例的注册和注销。他们也在每个集群节点上运行代理，来实现服务端发现路由器的功能。

HTTP 反向代理和负载据衡器（例如 NGINX）可以用于服务发现负载均衡器。服务注册表可以将路由信息推送到 NGINX，激活一个实时配置更新；例如，可以使用 Consul Template。NGINX Plus 支持[额外的动态重新配置机制](https://www.nginx.com/products/on-the-fly-reconfiguration/)，可以使用 DNS，将服务实例信息从注册表中拉下来，并且提供远程配置的 API。

在未来的博客中，我们还将深入探讨微服务其它特点。可以注册 NGINX 邮件列表来获得最新产品更新提示。

此篇其它博客译文参见如下地址：

*   [微服务架构的优势与不足](http://dockone.io/article/394)
*   [使用 API Gateway](http://dockone.io/article/482)
*   [深入微服务架构的进程间通信](http://dockone.io/article/549)

**原文链接：[Service Discovery in a Microservices Architecture](https://www.nginx.com/blog/service-discovery-in-a-microservices-architecture/) （翻译：杨峰 校对：宋喻）**