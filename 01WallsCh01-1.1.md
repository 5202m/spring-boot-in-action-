# 第1章 入门

>本章内容

>- Spring Boot简化Spring应用程序开发
- Spring Boot的基本特性
- Spring Boot工作区的设置

Spring Framework已有十余年的历史了，已成为Java应用程序开发框架的事实标准。在如此悠久的历史背景下，有人可能会认为Spring放慢了脚步，躺在了自己的荣誉簿上，再也做不出什么新鲜的东西，或者是让人激动的东西。甚至有人说，Spring是遗留项目，是时候去看看其他创新的东西了。

这些人说得不对。

Spring的生态圈里正在出现很多让人激动的新鲜事物，涉及的领域涵盖云计算、大数据、无模式的数据持久化、响应式编程以及客户端应用程序开发。

在过去的一年多时间里，最让人兴奋、回头率最高、最能改变游戏规则的东西，大概就是Spring Boot了。Spring Boot提供了一种新的编程范式，能在最小的阻力下开发Spring应用程序。有了它，你可以更加敏捷地开发Spring应用程序，专注于应用程序的功能，不用在Spring的配置上多花功夫，甚至完全不用配置。实际上，Spring Boot的一项重要工作就是让Spring不再成为你成功路上的绊脚石。

本书将探索Spring Boot开发的诸多方面，但在开始前，我们先大概了解一下Spring Boot的功能。

## 1.1 Spring风云再起

Spring诞生时是Java企业版（Java Enterprise Edition，JEE，也称J2EE）的轻量级代替品。无需开发重量级的Enterprise JavaBean（EJB），Spring为企业级Java开发提供了一种相对简单的方法，通过依赖注入和面向切面编程，用简单的Java对象（Plain Old Java Object，POJO）实现了EJB的功能。

虽然Spring的组件代码是轻量级的，但它的配置却是重量级的。一开始，Spring用XML配置，而且是很多XML配置。Spring 2.5引入了基于注解的组件扫描，这消除了大量针对应用程序自身组件的显式XML配置。Spring 3.0引入了基于Java的配置，这是一种类型安全的可重构配置方式，可以代替XML。

尽管如此，我们依旧没能逃脱配置的魔爪。开启某些Spring特性时，比如事务管理和Spring MVC，还是需要用XML或Java进行显式配置。启用第三方库时也需要显式配置，比如基于Thymeleaf的Web视图。配置Servlet和过滤器（比如Spring的`DispatcherServlet`）同样需要在web.xml或Servlet初始化代码里进行显式配置。组件扫描减少了配置量，Java配置让它看上去简洁不少，但Spring还是需要不少配置。

所有这些配置都代表了开发时的损耗。因为在思考Spring特性配置和解决业务问题之间需要进行思维切换，所以写配置挤占了写应用程序逻辑的时间。和所有框架一样，Spring实用，但与此同时它要求的回报也不少。

除此之外，项目的依赖管理也是件吃力不讨好的事情。决定项目里要用哪些库就已经够让人头痛的了，你还要知道这些库的哪个版本和其他库不会有冲突，这难题实在太棘手。

并且，依赖管理也是一种损耗，添加依赖不是写应用程序代码。一旦选错了依赖的版本，随之而来的不兼容问题毫无疑问会是生产力杀手。

Spring Boot让这一切成为了过去。

### 1.1.1 重新认识Spring

假设你受命用Spring开发一个简单的Hello World Web应用程序。你该做什么？我能想到一些基本的需要。

- 一个项目结构，其中有一个包含必要依赖的Maven或者Gradle构建文件；最起码要有Spring MVC和Servlet API这些依赖。
- 一个web.xml文件（或者一个`WebApplicationInitializer`实现），其中声明了Spring的`DispatcherServlet`。
- 一个启用了Spring MVC的Spring配置。
- 一个控制器类，以“Hello World”响应HTTP请求。
- 一个用于部署应用程序的Web应用服务器，比如Tomcat。

最让人难以接受的是，这份清单里只有一个东西是和Hello World功能相关的，即控制器，剩下的都是Spring开发的Web应用程序必需的通用样板。既然所有Spring Web应用程序都要用到它们，那为什么还要你来提供这些东西呢？

暂且假设控制器就是你所需要的一切，那么你会发现，代码清单1-1所示的基于Groovy的控制器类（虽然很简单）就是一个完整的Spring应用程序。

**代码清单1-1 一个完整的基于Groovy的Spring应用程序**
```
groovy
@RestController
class HelloController {
  @RequestMapping("/")
  def hello() {
    return "Hello World"
  }
}
```
这里没有配置，没有web.xml，没有构建说明，甚至没有应用服务器，但这就是整个应用程序了。Spring Boot会搞定执行应用程序所需的各种后勤工作，你只要搞定应用程序的代码就好。

假设你已经装好了Spring Boot的命令行界面（Command Line Interface，CLI），可以像下面这样在命令行里运行`HelloController`：
```
$ spring run HelloController.groovy
```
想必你已经注意到了，这里甚至没有编译代码，Spring Boot CLI可以运行未经编译的代码。

之所以选择用Groovy来写这个控制器示例，是因为Groovy语言的简洁与Spring Boot的简洁有异曲同工之妙。但Spring Boot并不强制要求使用Groovy。实际上，本书中的很多代码都是用Java写的，但在恰当的时候，偶尔也会出现一些Groovy代码。

不要客气，直接跳到1.2.1节吧，看看如何安装Spring Boot CLI，这样你就能试着编写这个小小的Web应用程序了。现在，你将看到Spring Boot的关键部分，看到它是如何改变Spring应用程序的开发方式的。

### 1.1.2 Spring Boot精要

Spring Boot将很多魔法带入了Spring应用程序的开发之中，其中最重要的是以下四个核心。

- *自动配置*：针对很多Spring应用程序常见的应用功能，Spring Boot能自动提供相关配置。
- *起步依赖*：告诉Spring Boot需要什么功能，它就能引入需要的库。
- *命令行界面*：这是Spring Boot的可选特性，借此你只需写代码就能完成完整的应用程序，无需传统项目构建。
- *Actuator*：让你能够深入运行中的Spring Boot应用程序，一探究竟。

每一个特性都在通过自己的方式简化Spring应用程序的开发。本书会探寻如何将它们发挥到极致，但就目前而言，先简单看看它们都提供了哪些功能吧。

####1.自动配置

在任何Spring应用程序的源代码里，你都会找到Java配置或XML配置（抑或两者皆有），它们为应用程序开启了特定的特性和功能。举个例子，如果你写过用JDBC访问关系型数据库的应用程序，那你一定在Spring应用程序上下文里配置过`JdbcTemplate`这个Bean。我打赌那段配置看起来是这样的：
```java
@Bean
public JdbcTemplate jdbcTemplate(DataSource dataSource) {
  return new JdbcTemplate(dataSource);
}
```
这段非常简单的Bean声明创建了一个`JdbcTemplate`的实例，注入了一个`DataSource`依赖。当然，这意味着你还需要配置一个`DataSource`的Bean，这样才能满足依赖。假设你将配置一个嵌入式H2数据库作为`DataSource` Bean，完成这个配置场景的代码大概是这样的：
```java
@Bean
public DataSource dataSource() {
  return new EmbeddedDatabaseBuilder()
          .setType(EmbeddedDatabaseType.H2)
          .addScripts('schema.sql', 'data.sql')
          .build();
}
```
这个Bean配置方法创建了一个嵌入式数据库，并指定在该数据库上执行两段SQL脚本。`build()`方法返回了一个指向该数据库的引用。

这两个Bean配置方法都不复杂，也不是很长，但它们只是典型Spring应用程序配置的一小部分。除此之外，还有无数Spring应用程序有着完全相同的方法。所有需要用到嵌入式数据库和`JdbcTemplate`的应用程序都会用到那些方法。简而言之，这就是一个样板配置。

既然它如此常见，那为什么还要你去写呢？

Spring Boot会为这些常见配置场景进行自动配置。如果Spring Boot在应用程序的Classpath里发现H2数据库的库，那么它就自动配置一个嵌入式H2数据库。如果在Classpath里发现`JdbcTemplate`，那么它还会为你配置一个`JdbcTemplate`的Bean。你无需操心那些Bean的配置，Spring Boot会做好准备，随时都能将其注入到你的Bean里。

Spring Boot的自动配置远不止嵌入式数据库和`JdbcTemplate`，它有大把的办法帮你减轻配置负担，这些自动配置涉及Java持久化API（Java Persistence API，JPA）、Thymeleaf模板、安全和Spring MVC。第2章会深入讨论自动配置这个话题。

####2.起步依赖

向项目中添加依赖是件富有挑战的事。你需要什么库？它的Group和Artifact是什么？你需要哪个版本？哪个版本不会和项目中的其他依赖发生冲突？

Spring Boot通过起步依赖为项目的依赖管理提供帮助。起步依赖其实就是特殊的Maven依赖和Gradle依赖，利用了传递依赖解析，把常用库聚合在一起，组成了几个为特定功能而定制的依赖。

举个例子，假设你正在用Spring MVC构造一个REST API，并将JSON（JavaScript Object Notation，JavaScript对象表示法）作为资源表述。此外，你还想运用遵循JSR-303规范的声明式校验，并使用嵌入式的Tomcat服务器来提供服务。要实现以上目标，你在Maven或Gradle里至少需要以下八个依赖：

- org.springframework:spring-core
- org.springframework:spring-web
- org.springframework:spring-webmvc
- com.fasterxml.jackson.core:jackson-databind
- org.hibernate:hibernate-validator
- org.apache.tomcat.embed:tomcat-embed-core
- org.apache.tomcat.embed:tomcat-embed-el
- org.apache.tomcat.embed:tomcat-embed-logging-juli

不过，如果打算利用Spring Boot的起步依赖，你只需添加Spring Boot的Web起步依赖（`org.springframework.boot:spring-boot-starter-web`）***{![Spring Boot的起步依赖基本都以`spring-boot-starter`打头，随后是直接代表其功能的名字，比如`web`、`test`，下文出现起步依赖的名字时，可能就直接用其前缀后的单词来表示了。——译者注]}***，仅此一个。它会根据依赖传递把其他所需依赖引入项目里，你都不用考虑它们。

比起减少依赖数量，起步依赖还引入了一些微妙的变化。向项目中添加了“web”起步依赖，实际上指定了应用程序所需的一类功能。因为应用是个Web应用程序，所以加入了“web”起步依赖。与之类似，如果应用程序要用到JPA持久化，那么就可以加入“jpa”起步依赖。如果需要安全功能，那就加入“security”起步依赖。简而言之，你不再需要考虑支持某种功能要用什么库了，引入相关起步依赖就行。

此外，Spring Boot的起步依赖还把你从“需要这些库的哪些版本”这个问题里解放了出来。起步依赖引入的库的版本都是经过测试的，因此你可以完全放心，它们之间不会出现不兼容的情况。

和自动配置一样，第2章就会深入讨论起步依赖。

####3.命令行界面

除了自动配置和起步依赖，Spring Boot还提供了一种很有意思的新方法，可以快速开发Spring应用程序。正如之前在1.1节里看到的那样，Spring Boot CLI让只写代码即可实现应用程序成为可能。

Spring Boot CLI利用了起步依赖和自动配置，让你专注于代码本身。不仅如此，你是否注意到代码清单1-1里没有`import`？CLI如何知道`RequestMapping`和`RestController`来自哪个包呢？说到这个问题，那些类最终又是怎么跑到Classpath里的呢？

说得简单一点，CLI能检测到你使用了哪些类，它知道要向Classpath中添加哪些起步依赖才能让它运转起来。一旦那些依赖出现在Classpath中，一系列自动配置就会接踵而来，确保启用`DispatcherServlet`和Spring MVC，这样控制器就能响应HTTP请求了。

Spring Boot CLI是Spring Boot的非必要组成部分。虽然它为Spring带来了惊人的力量，大大简化了开发，但也引入了一套不太常规的开发模型。要是这种开发模型与你的口味相去甚远，那也没关系，抛开CLI，你还是可以利用Spring Boot提供的其他东西。不过如果喜欢CLI，你一定想看看第5章，其中深入探讨了Spring Boot CLI。

####4.Actuator

Spring Boot的最后一块“拼图”是Actuator，其他几个部分旨在简化Spring开发，而Actuator则要提供在运行时检视应用程序内部情况的能力。安装了Actuator就能窥探应用程序的内部情况了，包括如下细节：

- Spring应用程序上下文里配置的Bean
- Spring Boot的自动配置做的决策
- 应用程序取到的环境变量、系统属性、配置属性和命令行参数
- 应用程序里线程的当前状态
- 应用程序最近处理过的HTTP请求的追踪情况
- 各种和内存用量、垃圾回收、Web请求以及数据源用量相关的指标

Actuator通过Web端点和shell界面向外界提供信息。如果要借助shell界面，你可以打开SSH（Secure Shell），登入运行中的应用程序，发送指令查看它的情况。

第7章会详细探索Actuator的功能。

### 1.1.3 Spring Boot不是什么

因为Spring Boot实在是太惊艳了，所以过去一年多的时间里有不少和它相关的言论。原先听到或看到的东西可能给你造成了一些误解，继续学习本书前应该先澄清这些误会。

首先，Spring Boot不是应用服务器。这个误解是这样产生的：Spring Boot可以把Web应用程序变为可自执行的JAR文件，不用部署到传统Java应用服务器里就能在命令行里运行。Spring Boot在应用程序里嵌入了一个Servlet容器（Tomcat、Jetty或Undertow），以此实现这一功能。但这是内嵌的Servlet容器提供的功能，不是Spring Boot实现的。

与之类似，Spring Boot也没有实现诸如JPA或JMS（Java Message Service，Java消息服务）之类的企业级Java规范。它的确支持不少企业级Java规范，但是要在Spring里自动配置支持那些特性的Bean。例如，Spring Boot没有实现JPA，不过它自动配置了某个JPA实现（比如Hibernate）的Bean，以此支持JPA。

最后，Spring Boot没有引入任何形式的代码生成，而是利用了Spring 4的条件化配置特性，以及Maven和Gradle提供的传递依赖解析，以此实现Spring应用程序上下文里的自动配置。

简而言之，从本质上来说，Spring Boot就是Spring，它做了那些没有它你自己也会去做的Spring Bean配置。谢天谢地，幸好有Spring，你不用再写这些样板配置了，可以专注于应用程序的逻辑，这些才是应用程序独一无二的东西。

现在，你应该对Spring Boot有了大概的认识，是时候构建你的第一个Spring Boot应用程序了。先从重要的事情开始，该怎么入手呢？

|[PREV](https://github.com/5202m/spring-boot-in-action-zh-cn/blob/master/00WallsFM.md)|[NEXT](https://github.com/5202m/spring-boot-in-action-zh-cn/blob/master/01WallsCh01-1.2.md)|
