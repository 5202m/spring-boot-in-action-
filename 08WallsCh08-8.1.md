# 第8章 部署Spring Boot应用程序

>本章内容

>-   部署WAR文件
- 数据库迁移
- 部署到云端

想一想你喜欢的动作电影。现在假设你要去电影院看这部电影，享受视听震撼。片中有高速追逐、爆炸和激战。好人还没战胜坏人，一切偏偏戛然而止。还没等影片里的冲突解决，电影院的灯亮了，大家都被领出门外。

虽然前面的铺垫很精彩，但电影的高潮才是最重要的。没有了它，就是为了动作而动作了。

现在，想象你正在开发应用程序，为解决某个业务问题投入了很多精力和创造力，但最终没能部署应用程序，没能让别人使用这个程序并乐在其中。当然，我们应用程序大多没有汽车追逐和爆炸（至少我希望是这样的），但一路上我们也会争分夺秒。当然，并非每行代码都为生产环境而写，但什么都不部署也挺让人失望的。

目前为止，我们的焦点都集中在使用Spring Boot的特性帮助大家开发应用程序。我们遇到了不少惊喜。但如果不越过终点线，应用程序没有部署，这一切都是徒劳。

在本章，我们会在使用Spring Boot开发应用程序的基础上更进一步，讨论如何部署那些应用程序。虽然这对部署过基于Java的应用程序的人来说并无特别之处，但Spring Boot和相关的Spring项目中有些独特的功能，基于这些功能我们可以让Spring Boot应用程序的部署变得与众不同。

实际上，大部分Java Web应用程序都以WAR文件的形式部署到应用服务器上。Spring Boot提供的部署方式则有所不同，后者在部署上提供了不少选择。在了解如何部署Spring Boot应用程序之前，让我们看看这些可选方式，找出能满足我们需求的那些选项。

## 8.1 衡量多种部署方式

Spring Boot应用程序有多种构建和运行方式，其中一些你已经使用过了。

- 在IDE中运行应用程序（涉及Spring ToolSuite或IntelliJ IDEA）。
- 使用Maven的`spring-boot:run`或Gradle的`bootRun`，在命令行里运行。
- 使用Maven或Gradle生成可运行的JAR文件，随后在命令行中运行。
- 使用Spring Boot CLI在命令行中运行Groovy脚本。
- 使用Spring Boot CLI来生成可运行的JAR文件，随后在命令行中运行。

这些选项每一个都适合运行正在开发的应用程序。但是，如果要将应用程序部署到生产环境或其他非开发环境中，又该怎么办呢？

虽然这些选项看起来没有一个能将应用部署于非开发环境，但事实上，它们之中只有一个选项不可用于生产环境——在IDE中运行应用显然不可取。可运行的JAR文件和Spring Boot CLI还是可以考虑的，两者还可以很好地将应用程序部署到云环境里。

也许你很想知道如何把Spring Boot应用程序部署到一个更加传统的应用服务器环境里，比如Tomcat、WebSphere或WebLogic。在这些情境中，可执行JAR文件和Groovy代码不适用。针对应用服务器的部署，你需要将应用程序打包成一个WAR文件。

实际上，Spring Boot应用程序可以用多种方式打包，详见表8-1。

__表8-1 Spring Boot部署选项__

| 部署产物 | 产生方式 | 目标环境 |
|---------|---------|----------|
| Groovy源码 | 手写 | Cloud Foundry及容器部署，比如Docker |
| 可执行JAR | Maven、Gradle或Spring Boot CLI | 云环境，包括Cloud Foundry和Heroku，还有容器部署，比如Docker |
| WAR | Maven或Gradle | Java应用服务器或云环境，比如Cloud Foundry |

如你所见，在做最终选择时需要考虑目标环境。如果要将应用程序部署到自己数据中心的Tomcat服务器上，WAR文件就是你的选择。另一方面，如果要部署到Cloud Foundry，你可以使用表里列出的各种选项。

在本章，我们将关注以下这些选项。

- 向Java应用服务器里部署WAR文件。
- 向Cloud Foundry里部署可执行JAR文件。
- 向Heroku里部署可执行JAR文件（构建过程是由Heroku执行的）。

探索这些场景的时候，我们还要处理一件事。在开发应用程序时我们使用了嵌入式的H2数据库，现在得把它替换成生产环境所需的数据库了。

首先，让我们看看如何将阅读列表应用程序构建为WAR文件。这样才能把它部署到Java应用服务器里，比如Tomcat、WebSphere或WebLogic。

| [PREV](https://github.com/5202m/spring-boot-in-action-zh-cn/blob/master/07WallsCh07-7.5.md) | [NEXT](https://github.com/5202m/spring-boot-in-action-zh-cn/blob/master/08WallsCh08-8.2.md) |
