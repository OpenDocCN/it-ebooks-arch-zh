# 全栈篇： 架构设计

### 博客

我尚不属于那些技术特别好的人——我只是广度特别广，从拿电烙铁到所谓的大数据。不过相比于所谓的大数据，我想我更擅长于焊电路板，笑~~。由于并非毕业于计算机专业，毕业前的实习过程中，我发现在某些特殊领域的技术比不上科班毕业的人，这意味着需要更多的学习。但是后来受益于工作近两年来从没有加班过，朝九晚六的生活带来了大量的学习时间。在这个漫长的追赶过程中，我发现开发博客相关的应用带来了很大的进步。

### 技术组成

So，在这个博客里会有三个用户来源，Web > 公众号 > App。

在网页上，每天大概会 400 个 PV，其中大部分是来自 Google、百度，接着就是偶尔推送的公众号，最后就是只有我一个人用的 APP。。。

> Web 架构

服务器：

1.  Nginx(含 Nginx HTTP 2.0、PageSpeed 插件)
2.  Gunicorn(2 Workers)
3.  New Relic(性能监测)

DevOps:

1.  Farbic（自动部署）

Web 应用后台：

1.  Mezzaine（基于 Django 的 CMS）
2.  REST Framework (API)
3.  REST Framework JWT (JSON Web Token)
4.  Wechat Python SDK
5.  Mezzanine Pagedown （Markdown

Web 应用前台:

1.  Material Design Lite (用户)
2.  BootStrap (后台)
3.  jQuery + jQuery.autocomplete + jquery.githubRepoWidget
4.  HighLight.js
5.  Angluar.js
6.  Backbone (已不维护)

移动端:

1.  Ionic
2.  Angular + ngCordova
3.  Cordova
4.  highlightjs
5.  showdown.js(Markdown Render)
6.  Angular Messages + Angular-elastic

微信端:

1.  Wechat-Python-SDK

That’s All…

### 前后台分离

### API

在构建 SPA 的时候，做了一些 API，然后就有了一个 Auto Sugget 的功能：

![Auto Suggest](img/2015-12-28_5680cac447393.jpg)

Auto Suggest

或者说，它是一个 Auto Complete，可以直接借助于 jQuery AutoComplete 插件。

或许你已经猜到了，既然我们已经有博客详情页和列表页的 API，并且我们也已经有了 Auto Suggestion API。那么，我们就可以有一个 APP 了。

### APP

偶然间发现了 Ionic 框架，它等于 = Angluar + Cordova。于是，在测试 Google Indexing 的时候，花了一个晚上做了博客的 APP。

![Blog App](img/2015-12-28_5680cac45621d.jpg)

Blog App

我们可以在上面做搜索，搜索的时候也会有 Auto Suggestion。上面的注销意味着它有登录功能，而 Hybird App 的登录通常可以借用于 JSON Web Token。即在第一次登录的时候生成一个 Token，之后的请求，如发博客、创建事件，都可以用这个 Token 来进行，直到 Token 过期。如果你是第一次在手机上访问，也许你会遇到这个没有节操的广告：

![Install Phodal Blog App](img/2015-12-28_5680cac467064.jpg)

Install Phodal Blog App

然并卵，作为我的第七个 Hybird 应用，它只发布在 Google Play 上——因为不需要审核。

随后，我意识到了我需要将我的博客推送给读者，但是需要一个渠道。

### 微信公众平台

借助于 Wechat-Python-SDK，花了一个下午做了一个基础的公众平台。除了可以查询最新的博客和搜索，它的主要作用就是让我发我的博客了。

对了，如果你用 Python 写代码，可以试试 PyCharm。除了 WebStorm 以外，我最喜欢的 IDE。因为 WebStorm 一直在与时俱进。