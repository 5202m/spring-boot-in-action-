# 第7章 深入Actuator

>本章内容

> * Actuator Web端点
* 调整Actuator
* 通过shell连入运行中的应用程序
* 保护Actuator

你有没有猜过包好的礼物盒里装的是什么东西？你会摇一摇，掂一掂，量一量，你甚至会执着于里面到底有什么。但打开盒子那一刻前，你没办法确认里面是什么。

运行中的应用程序就像礼物盒。你可以刺探它，作出合理的推测，猜测它的运行情况。但如何了解真实的情况呢？有没有一种办法能让你深入应用程序内部一窥究竟，了解它的行为，检查它的健康状况，甚至触发一些操作来影响应用程序呢？

在本章中，我们将了解Spring Boot的Actuator。它提供了很多生产级的特性，比如监控和度量Spring Boot应用程序。Actuator的这些特性可以通过众多REST端点、远程shell和JMX获得。我们先来看看Actuator的REST端点，这种最为人所熟知的使用方式提供了最完整的功能。

## 7.1 揭秘Actuator的端点

Spring Boot Actuator的关键特性是在应用程序里提供众多Web端点，通过它们了解应用程序运行时的内部状况。有了Actuator，你可以知道Bean在Spring应用程序上下文里是如何组装在一起的，掌握应用程序可以获取的环境属性信息，获取运行时度量信息的快照……

Actuator提供了13个端点，具体如表7-1所示。

__表7-1 Actuator的端点__

| HTTP方法 | 路径              | 描述                                                           |
|----------|-------------------|---------------------------------------------------------------|
| `GET`      | /autoconfig     | 提供了一份自动配置报告，记录哪些自动配置条件通过了，哪些没通过  |
| `GET`      | /configprops    | 描述配置属性（包含默认值）如何注入Bean                |
| `GET`      | /beans          | 描述应用程序上下文里全部的Bean，以及它们的关系             |
| `GET`      | /dump           | 获取线程活动的快照                                            |
| `GET`      | /env            | 获取全部环境属性                                              |
| `GET`      | /env/{name}     | 根据名称获取特定的环境属性值                                   |
| `GET`      | /health         | 报告应用程序的健康指标，这些值由`HealthIndicator`的实现类提供    |
| `GET`      | /info           | 获取应用程序的定制信息，这些信息由`info`打头的属性提供           |
| `GET`      | /mappings       | 描述全部的URI路径，以及它们和控制器（包含Actuator端点）的映射关系 |
| `GET`      | /metrics        | 报告各种应用程序度量信息，比如内存用量和HTTP请求计数             |
| `GET`      | /metrics/{name} | 报告指定名称的应用程序度量值                                    |
| `GET`      | /shutdown       | 关闭应用程序，要求`endpoints.shutdown.enabled`设置为`true`     |
| `GET`      | /trace          | 提供基本的HTTP请求跟踪信息（时间戳、HTTP头等）                  |

要启用Actuator的端点，只需在项目中引入Actuator的起步依赖即可。在Gradle构建说明文件里，这个依赖是这样的：

```
compile 'org.springframework.boot:spring-boot-starter-actuator'
```

对于Maven项目，引入的依赖是这样的：

```
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

亦或你使用Spring Boot CLI，可以使用如下`@Grab`注解：

```
@Grab('spring-boot-starter-actuator')
```

无论Actuator是如何添加的，在应用程序运行时自动配置都会生效。Actuator会开启。

表7-1中的端点可以分为三大类：配置端点、度量端点和其他端点。让我们分别了解一下这些端点，从提供应用程序配置信息的端点看起。

### 7.1.1 查看配置明细

关于Spring组件扫描和自动织入，最常遭人抱怨的问题之一就是很难看到应用程序中的组件是如何装配起来的。Spring Boot自动配置让这个问题变得更严重，因为Spring的配置更少了。在有显式配置的情况下，你至少还能看到XML文件或者配置类，对Spring应用程序上下文里的Bean关系有个大概的了解。

我个人从不担心这个问题。也许是因为我意识到，在Spring出现之前，根本就没有应用程序组件的映射关系。

但是，如果你担心自动配置隐藏了Spring应用程序上下文中Bean的装配细节，那么我要告诉你一个好消息！Actuator有一些端点不仅可以显示组件映射关系，还可以告诉你自动配置在配置Spring应用程序上下文时做了哪些决策。

#### 1.获得Bean装配报告

要了解应用程序中Spring上下文的情况，最重要的端点就是/beans。它会返回一个JSON文档，描述上下文里每个Bean的情况，包括其Java类型以及注入的其他Bean。向/beans（在本地运行时是[http://localhost:8080/beans](http://localhost:8080/beans)）发起`GET`请求后，你会看到与代码清单7-1示例类似的信息。

__代码清单7-1 /beans端点提供的Spring应用程序上下文Bean信息__

```
[
  {
    "beans": [
      {
        "bean": "application",
        "dependencies": [],
        "resource": "null",
        "scope": "singleton",
        "type": "readinglist.Application$$EnhancerBySpringCGLIB$$f363c202"
      },
      {
        "bean": "amazonProperties",
        "dependencies": [],
        "resource": "URL [jar:file:/../readinglist-0.0.1-SNAPSHOT.jar!
                                      /readinglist/AmazonProperties.class]",
        "scope": "singleton",
        "type": "readinglist.AmazonProperties"
      },
      {
        "bean": "readingListController",
        "dependencies": [
          "readingListRepository",
          "amazonProperties"
        ],
        "resource": "URL [jar:file:/../readinglist-0.0.1-SNAPSHOT.jar!
                               /readinglist/ReadingListController.class]",
        "scope": "singleton",
        "type": "readinglist.ReadingListController"
      },
      {
        "bean": "readerRepository",
        "dependencies": [
          "(inner bean)#219df4f5",
          "(inner bean)#2c0e7419",
          "(inner bean)#7d86037b",
          "jpaMappingContext"
        ],
        "resource": "null",
        "scope": "singleton",
        "type": "readinglist.ReaderRepository"
      },
      {
        "bean": "readingListRepository",
        "dependencies": [
          "(inner bean)#98ce66",
          "(inner bean)#1fd7add0",
          "(inner bean)#59faabb2",
          "jpaMappingContext"
        ],
        "resource": "null",
        "scope": "singleton",
        "type": "readinglist.ReadingListRepository"
      },
      ...
    ],
    "context": "application",
    "parent": null
  }
]
```
>文字

>Bean ID：Bean ID

>Resource file：资源文件

>Dependencies：依赖

>Bean scope：Bean作用域

>Java type：Java类型

代码清单7-1是阅读列表应用程序Bean信息的一个片段。如你所见，所有的Bean条目都有五类信息。

* `bean`：Spring应用程序上下文中的Bean名称或ID。
* `resource`：.class文件的物理位置，通常是一个URL，指向构建出的JAR文件。这会随着应用程序的构建和运行方式发生变化。
* `dependencies`：当前Bean注入的Bean ID列表。
* `scope`：Bean的作用域（通常是单例，这也是默认作用域）。
* `type`：Bean的Java类型。

虽然Bean报告不用具体绘图告诉你Bean是如何装配的（例如，通过属性或构造方法），但它帮你直观地了解了应用程序上下文中Bean的关系。实际上，写出一个工具，把Bean报告处理一下，用图形化的方式来展现Bean关系，这并不难。请注意，完整的Bean报告会包含很多Bean，还有很多自动配置的Bean，画出来的图会非常复杂。

#### 2.详解自动配置

/beans端点产生的报告能告诉你Spring应用程序上下文里都有哪些Bean。/autoconfig端点能告诉你为什么会有这个Bean，或者为什么没有这个Bean。

正如第2章里说的，Spring Boot自动配置构建于Spring的条件化配置之上。它提供了众多带有`@Conditional`注解的配置类，根据条件决定是否要自动配置这些Bean。/autoconfig端点提供了一个报告，列出了计算过的所有条件，根据条件是否通过进行分组。

代码清单7-2是阅读列表应用程序自动配置报告里的一个片段，里面有一个通过的条件，还有一个没通过的条件。

__代码清单7-2 阅读列表应用程序的自动配置报告__

```
{
  "positiveMatches": {
    ...
    "DataSourceAutoConfiguration.JdbcTemplateConfiguration
                                                    #jdbcTemplate": [
      {
        "condition": "OnBeanCondition",
        "message": "@ConditionalOnMissingBean (types:
            org.springframework.jdbc.core.JdbcOperations;
            SearchStrategy: all) found no beans"
      }
    ],
    ...
  },
  "negativeMatches": {
    "ActiveMQAutoConfiguration": [
      {
        "condition": "OnClassCondition",
        "message": "required @ConditionalOnClass classes not found:
           javax.jms.ConnectionFactory,org.apache.activemq
           .ActiveMQConnectionFactory"
      }
    ],
    ...
  }
}
```
>文字

>Successful conditions：成功条件

>Failed conditions：失败条件

在`positiveMatches`里，你会看到一个条件，决定Spring Boot是否自动配置`JdbcTemplate` Bean。匹配到的名字是`DataSourceAutoConfiguration.JdbcTemplateConfiguration#jdbcTemplate`，这是运用了条件的具体配置类。条件类型是`OnBeanCondition`，意味着条件的输出是由某个Bean的存在与否来决定的。在本例中，`message`属性已经清晰地表明了该条件是检查是否有`JdbcOperations`类型（`JbdcTemplate`实现了该接口）的Bean存在。如果没有配置这种Bean，则条件成立，创建一个`JdbcTemplate` Bean。

与之类似，在`negativeMatches`里，有一个条件决定了是否要配置ActiveMQ。这是一个`OnClassCondition`，会检查Classpath里是否存在`ActiveMQConnectionFactory`。因为Classpath里没有这个类，条件不成立，所以不会自动配置ActiveMQ。

#### 3.查看配置属性

除了要知道应用程序的Bean是如何装配的，你可能还对能获取哪些环境属性，哪些配置属性注入了Bean里感兴趣。

/env端点会生成应用程序可用的所有环境属性的列表，无论这些属性是否用到。这其中包括环境变量、JVM属性、命令行参数，以及applicaition.properties或application.yml文件提供的属性。

代码清单7-3的示例代码是/env端点获取信息的一个片段。

__代码清单7-3 /env端点会报告所有可用的属性__

```
{
  "applicationConfig: [classpath:/application.yml]": {
    "amazon.associate_id": "habuma-20",
    "error.whitelabel.enabled": false,
    "logging.level.root": "INFO"
  },
  "profiles": [],
  "servletContextInitParams": {},
  "systemEnvironment": {
    "BOOK_HOME": "/Users/habuma/Projects/BookProjects/walls6",
    "GRADLE_HOME": "/Users/habuma/.sdkman/gradle/current",
    "GRAILS_HOME": "/Users/habuma/.sdkman/grails/current",
    "GROOVY_HOME": "/Users/habuma/.sdkman/groovy/current",
    ...
  },
  "systemProperties": {
    "PID": "682",
    "file.encoding": "UTF-8",
    "file.encoding.pkg": "sun.io",
    "file.separator": "/",
    ...
  }
}
```
>文字

>Application properties：应用属性

>Environment variables：环境变量

>JVM system properties：JVM系统属性

基本上，任何能给Spring Boot应用程序提供属性的属性源都会列在/env的结果里，同时会显示具体的属性。

代码清单7-3中的属性来源有很多，包括应用程序配置（application.yml）、Spring Profile、Servlet上下文初始化参数、系统环境变量和JVM系统属性。（本例中没有Profile和Servlet上下文初始化参数。）

属性常用来提供诸如数据库或API密码之类的敏感信息。为了避免此类信息暴露到/env里，所有名为password、secret、key（或者名字中最后一段是这些）的属性在/env里都会加上“*”。举个例子，如果有一个属性名字是database.password，那么它在/env中的显示效果是这样的：
```
"database.password":"******"
```
/env端点还能用来获取单个属性的值，只需要在请求时在/env后加上属性名即可。举例来说，对阅读列表应用程序发起`/env/amazon.associate_id`请求，获得的结果是habuma-20（纯文本形式）。

回想第3章，通过`@ConfigurationProperties`注解可以很方便地使用这些环境属性。这些环境属性会注入带有`@ConfigurationProperties`注解的Bean的实例属性。/configprops端点会生成一个报告，说明这些属性是如何进行设置的（注入或其他方式）。代码清单7-4是阅读列表应用程序的配置属性报告片段。

__代码清单7-4 配置属性报告__

```
{
  "amazonProperties": {
    "prefix": "amazon",
    "properties": {
      "associateId": "habuma-20"
    }
  },
  ...
  "serverProperties": {
    "prefix": "server",
    "properties": {
      "address": null,
      "contextPath": null,
      "port": null,
      "servletPath": "/",
      "sessionTimeout": null,
      "ssl": null,
      "tomcat": {
        "accessLogEnabled": false,
        "accessLogPattern": null,
        "backgroundProcessorDelay": 30,
        "basedir": null,
        "compressableMimeTypes": "text/html,text/xml,text/plain",
        "compression": "off",
        "maxHttpHeaderSize": 0,
        "maxThreads": 0,
        "portHeader": null,
        "protocolHeader": null,
        "remoteIpHeader": null,
        "uriEncoding": null
      },
      ...
    }
  },
  ...
}
```
>文字

>Amazon configuration：Amazon配置

>Server configuration：服务器配置

片段中的第一个内容是我们在第3章里创建的`amazonProperties` Bean。报告显示它添加了`@ConfigurationProperties`注解，前缀为`amazon`。`associateId`属性设置为habuma-20。这是因为在application.yml里，我们把`amazon.associateId`属性设置成了habuma-20。

你还会看到一个`serverProperties`条目（前缀是`server`），还有一些属性。它们都有默认值，你也可以通过设置`server`前缀的属性来改变这些值。举例来说，你可以通过设置`server.port`属性来修改服务器监听的端口。

除了展现运行中应用程序的配置属性如何设置，这个报告也能作为一个快速参考指南，告诉你有哪些属性可以设置。例如，如果你不清楚怎么设置嵌入式Tomcat服务器的最大线程数，可以看一下配置属性报告，里面会有一条`server.tomcat.maxThreads`，这就是你要找的属性。

#### 4.生成端点到控制器的映射

在应用程序相对较小的时候，很容易搞清楚控制器都映射到了哪些端点上。如果Web界面的控制器和请求处理方法数量多，那最好能有一个列表，罗列出应用程序发布的全部端点。

/mappings端点就提供了这么一个列表。代码清单7-5是阅读列表应用程序的映射报告片段。

**代码清单7-5 阅读列表应用程序的控制器/端点映射**

```
{
  ...

  "{[/],methods=[GET],params=[],headers=[],consumes=[],produces=[],
                                                       custom=[]}": {
    "bean": "requestMappingHandlerMapping",
    "method": "public java.lang.String readinglist.ReadingListController.
              readersBooks(readinglist.Reader,org.springframework.ui.Model)"
  },
  "{[/],methods=[POST],params=[],headers=[],consumes=[],produces=[],
                                                        custom=[]}": {
    "bean": "requestMappingHandlerMapping",
    "method": "public java.lang.String readinglist.ReadingListController
                            .addToReadingList(readinglist.Reader,readinglist.
     Book)"
  },
  "{[/autoconfig],methods=[GET],params=[],headers=[],consumes=[]
                                          ,produces=[],custom=[]}": {
    "bean": "endpointHandlerMapping",
    "method": "public java.lang.Object org.springframework.boot
                   .actuate.endpoint.mvc.EndpointMvcAdapter.invoke()"
  },
  ...
}
```
>文字

>ReadingListController mappings：`ReadingListController`映射

>Auto-configuration report mapping：自动配置报告的映射

这里我们可以看到不少端点的映射。每个映射的键都是一个字符串，其内容就是Spring MVC的`@RequestMapping`注解上设置的属性。实际上，这个字符串能让你清晰地了解控制器是如何映射的，哪怕不看源代码。每个映射的值都有两个属性：`bean`和`method`。`bean`属性标识了Spring Bean的名字，映射源自这个Bean。`method`属性是映射对应方法的全限定方法签名。

头两个映射关乎应用程序中`ReadingListController`的请求如何处理。第一个表明`readersBooks()`方法处理根路径（“/”）的HTTP `GET`请求。第二个表明`POST`请求映射到`addToReadingList()`方法上。

接下来的映射是Actuator提供的端点。/autoconfig端点的HTTP `GET`请求由Spring Boot的`EndpointMvcAdapter`类的`invoke()`方法来处理。当然，还有很多其他Actuator的端点没有列在代码清单7-5里，这种省略完全是为了简化代码示例。

Actuator的配置端点能很方便地让你了解应用程序是如何配置的。能看到应用程序在运行时究竟发生了什么，这很有趣、很实用。度量端点能展示应用程序运行时内部状况的快照。

### 7.1.2 运行时度量

你到医生那里体检时，会做一系列检查来了解身体状况。有一些重要的项目永远不会变，比如血型。这类测试能让医生了解你身体的一贯情况。其他测试让医生掌握你接受检查时的身体状况。你的心律、血压和胆固醇水平有助于医生评估你的健康。这些指标都是临时的，很可能随时间发生变化，但它们同样是很有帮助的运行时指标。

与之类似，对运行时度量情况做一个快照，这对评估应用程序的健康情况很有帮助。Actuator提供了一系列端点，让你能在运行时快速检查应用程序。让我们来了解一下这些端点，从`/metrics`开始。

#### 1.查看应用程序的度量值

关于运行中的应用程序，有很多有趣而且有用的信息。举个例子，了解应用程序的内存情况（可用或空闲）有助于决定给JVM分配多少内存。对Web应用程序而言，不用查看Web服务器日志，如果请求失败或者是耗时太长，就可以大概知道内存的情况了。

运行中的应用程序有诸多计数器和度量器，/metrics端点提供了这些东西的快照。代码清单7-6是/metrics端点输出内容的示例。

__代码清单7-6 /metrics端点提供了很多有用的运行时数据__

```
{
  mem: 198144,
  mem.free: 144029,
  processors: 8,
  uptime: 1887794,
  instance.uptime: 1871237,
  systemload.average: 1.33251953125,
  heap.committed: 198144,
  heap.init: 131072,
  heap.used: 54114,
  heap: 1864192,
  threads.peak: 21,
  threads.daemon: 19,
  threads: 21,
  classes: 9749,
  classes.loaded: 9749,
  classes.unloaded: 0,
  gc.ps_scavenge.count: 22,
  gc.ps_scavenge.time: 122,
  gc.ps_marksweep.count: 2,
  gc.ps_marksweep.time: 156,
  httpsessions.max: -1,
  httpsessions.active: 1,
  datasource.primary.active: 0,
  datasource.primary.usage: 0,
  counter.status.200.beans: 1,
  counter.status.200.env: 1,
  counter.status.200.login: 3,
  counter.status.200.metrics: 2,
  counter.status.200.root: 6,
  counter.status.200.star-star: 9,
  counter.status.302.login: 3,
  counter.status.302.logout: 1,
  counter.status.302.root: 5,
  gauge.response.beans: 169,
  gauge.response.env: 165,
  gauge.response.login: 3,
  gauge.response.logout: 0,
  gauge.response.metrics: 2,
  gauge.response.root: 11,
  gauge.response.star-star: 2
}
```

如你所见，/metrics端点提供了很多信息，逐行查看这些度量值太麻烦。表7-2根据所提供信息的类型对它们做了个分类。

__表7-2 /metrics端点报告的度量值和计数器__

| 分类      | 前缀              | 报告内容                                            |
|----------|-------------------|---------------------------------------------------|
| 垃圾收集器 | `gc.*` | 已经发生过的垃圾收集次数，以及垃圾收集所耗费的时间，适用于标记-清理垃圾收集器和并行垃圾收集器（数据源自`java.lang.management.GarbageCollectorMXBean`） |
| 内存 | `mem.*` | 分配给应用程序的内存数量和空闲的内存数量（数据源自`java.lang.Runtime`） |
| 堆 | `heap.*` | 当前内存用量（数据源自`java.lang.management.MemoryUsage`） |
| 类加载器 | `classes.*` | JVM类加载器加载与卸载的类的数量（数据源自`java.lang.management.ClassLoadingMXBean`） |
| 系统 | `processors`、`uptime instance.uptime`、`systemload.average` | 系统信息，例如处理器数量（数据源自`java.lang.Runtime`）、运行时间（数据源自`java.lang.management.RuntimeMXBean`）、平均负载（数据源自`java.lang.management.OperatingSystemMXBean`） |
| 线程池 | `threads.*` | 线程、守护线程的数量，以及JVM启动后的线程数量峰值（数据源自`java.lang .management.ThreadMXBean`） |
| 数据源 | `datasource.*` | 数据源连接的数量（源自数据源的元数据，仅当Spring应用程序上下文里存在`DataSource` Bean的时候才会有这个信息） |
| Tomcat会话 | `httpsessions.*` | Tomcat的活跃会话数和最大会话数（数据源自嵌入式Tomcat的Bean，仅在使用嵌入式Tomcat服务器运行应用程序时才有这个信息） |
| HTTP | `counter.status.*`、 `gauge.response.*` | 多种应用程序服务HTTP请求的度量值与计数器 |

请注意，这里的一些度量值，比如数据源和Tomcat会话，仅在应用程序中运行特定组件时才有数据。你还可以注册自己的度量信息，7.4.3节里会提到这一点。

HTTP的计数器和度量值需要做一点说明。counter.status后的值是HTTP状态码，随后是所请求的路径。举个例子，`counter.status.200.metrics`表明/metrics端点返回200（OK）状态码的次数。

HTTP的度量信息在结构上也差不多，却在报告另一类信息。它们全部用gauge.response开头，表明这是HTTP响应的度量信息。前缀后是对应的路径。度量值是以毫秒为单位的时间，反映了最近处理该路径请求的耗时。举个例子，代码清单7-6里的`gauge.response.beans`说明上一次请求耗时169毫秒。

这里还有几个特殊的值需要注意。root路径指向的是根路径或`/`。star-star代表了那些Spring认为是静态资源的路径，包括图片、JavaScript和样式表，其中还包含了那些找不到的资源。这就是为什么你经常会看到`counter.status.404.star-star`，这是返回了HTTP 404（NOT FOUND）状态的请求数。

/metrics端点会返回所有的可用度量值，但你也可能只对某个值感兴趣。要获取单个值，请求时可以在URL后加上对应的键名。例如，要查看空闲内存大小，可以向/metrics/mem.free发一个`GET`请求：

```
$ curl localhost:8080/metrics/mem.free
144029
```

要知道，虽然响应里的`Content-Type`头设置为application/json;charset=UTF-8，但实际/metrics/{name}的结果是文本格式的。因此，如果需要的话，你也可以把它视为JSON来处理。

#### 2.追踪Web请求

尽管/metrics端点提供了一些针对Web请求的基本计数器和计时器，但那些度量值缺少详细信息。知道所处理请求的更多信息是很有帮助的，尤其是在调试时，所以就有了/trace这个端点。

/trace端点能报告所有Web请求的详细信息，包括请求方法、路径、时间戳以及请求和响应的头信息。代码清单7-7是/trace输出的一个片段，其中包含了整个请求跟踪项。

__代码清单7-7 /trace端点会记录下Web请求的细节__

```
[
  ...
  {
    "timestamp": 1426378239775,
    "info": {
      "method": "GET",
      "path": "/metrics",
      "headers": {
        "request": {
          "accept": "*/*",
          "host": "localhost:8080",
          "user-agent": "curl/7.37.1"
        },
        "response": {
          "X-Content-Type-Options": "nosniff",
          "X-XSS-Protection": "1; mode=block",
          "Cache-Control":
                    "no-cache, no-store, max-age=0, must-revalidate",
          "Pragma": "no-cache",
          "Expires": "0",
          "X-Frame-Options": "DENY",
          "X-Application-Context": "application",
          "Content-Type": "application/json;charset=UTF-8",
          "Transfer-Encoding": "chunked",
          "Date": "Sun, 15 Mar 2015 00:10:39 GMT",
          "status": "200"
        }
      }
    }
  }
]
```

正如`method`和`path`属性所示，你可以看到这个跟踪项是一个针对/metrics的请求。`timestamp`属性（以及响应中的`Date`头）告诉了你请求的处理时间。`headers`属性的内容是请求和响应中所携带的头信息。

虽然代码清单7-7里只显示了一条跟踪项，但/trace端点实际能显示最近100个请求的信息，包含对/trace自己的请求。它在内存里维护了一个跟踪库。稍后在7.4.4节里，你会看到如何创建一个自定义的跟踪库实现，以便将请求的跟踪持久化。

#### 3.导出线程活动

在确认应用程序运行情况时，除了跟踪请求，了解线程活动也会很有帮助。/dump端点会生成当前线程活动的快照。

完整的线程导出报告里会包含应用程序的每个线程。为了节省空间，代码清单7-8里只放了一个线程的内容片段。如你所见，其中包含很多线程的特定信息，还有线程相关的阻塞和锁状态。本例中，还有一个跟踪栈（stack trace），表明这是一个Tomcat容器线程。

__代码清单7-8 /dump端点提供了应用程序线程的快照__

```
[
  {
    "threadName": "container-0",
    "threadId": 19,
    "blockedTime": -1,
    "blockedCount": 0,
    "waitedTime": -1,
    "waitedCount": 64,
    "lockName": null,
    "lockOwnerId": -1,
    "lockOwnerName": null,
    "inNative": false,
    "suspended": false,
    "threadState": "TIMED_WAITING",
    "stackTrace": [
      {
        "className": "java.lang.Thread",
        "fileName": "Thread.java",
        "lineNumber": -2,
        "methodName": "sleep",
        "nativeMethod": true
      },
      {
        "nativeMethod": false
      },
      {
        "className": "org.springframework.boot.context.embedded.
                            tomcat.TomcatEmbeddedServletContainer$1",
        "fileName": "TomcatEmbeddedServletContainer.java",
        "lineNumber": 139,
        "methodName": "run",
        "nativeMethod": false
      }
    ],
    "lockedMonitors": [],
    "lockedSynchronizers": [],
    "lockInfo": null
  },
  ...
]
```

#### 4.监控应用程序健康情况

如果你想知道自己的应用程序是否在运行，可以直接访问/health端点。在最简单的情况下，该端点会显示一个简单的JSON，内容如下：

```
{"status":"UP"}
```

`status`属性显示了应用程序在运行中。当然，它的确在运行，此处的响应无关紧要，任何输出都说明这个应用程序在运行。但`/health`端点可以输出的信息远远不止简单的`UP`状态。

/health端点输出的某些信息可能涉及内容，因此对未经授权的请求只能提供简单的健康状态。如果经过身份验证（比如你已经登录了），则可以提供更多信息。下面是阅读列表应用程序一些健康信息的示例：

```
{
  "status":"UP",
  "diskSpace": {
    "status":"UP",
    "free":377423302656,
    "threshold":10485760
  },
  "db":{
    "status":"UP",
    "database":"H2",
    "hello":1
  }
}
```

除了基本的健康状态，可用的磁盘空间以及应用程序正在使用的数据库状态也可以看到。

/health端点所提供的所有信息都是由一个或多个健康指示器提供的。表7-3列出了Spring Boot自带的健康指示器。

__表7-3 Spring Boot自带的健康指示器__

| 健康指示器 | 键 | 报告内容 |
|----------|-----|---------|
|` ApplicationHealthIndicator` | `none` | 永远为`UP` |
| `DataSourceHealthIndicator` | `db` | 如果数据库能连上，则内容是`UP`和数据库类型；否则为`DOWN` |
| `DiskSpaceHealthIndicator` | `diskSpace` | 如果可用空间大于阈值，则内容为`UP`和可用磁盘空间；如果空间不足则为`DOWN` |
| `JmsHealthIndicator` | `jms` | 如果能连上消息代理，则内容为`UP`和JMS提供方的名称；否则为`DOWN` |
| `MailHealthIndicator` | `mail` | 如果能连上邮件服务器，则内容为`UP`和邮件服务器主机和端口；否则为`DOWN` |
| `MongoHealthIndicator` | `mongo` | 如果能连上MongoDB服务器，则内容为`UP`和MongoDB服务器版本；否则为`DOWN` |
| `RabbitHealthIndicator` | `rabbit` | 如果能连上RabbitMQ服务器，则内容为`UP`和版本号；否则为`DOWN` |
| `RedisHealthIndicator` | `redis` | 如果能连上服务器，则内容为`UP`和Redis服务器版本；否则为`DOWN` |
| `SolrHealthIndicator` | `solr` | 如果能连上Solr服务器，则内容为`UP`；否则为`DOWN` |

这些健康指示器会按需自动配置。举例来说，如果Classpath里有`javax.sql.DataSource`，则会自动配置`DataSourceHealthIndicator`。`ApplicationHealthIndicator`和`DiskSpaceHealthIndicator`则会一直配置着。

除了这些自带的健康指示器，你还会在7.4.5节里看到如何创建自定义健康指示器。

### 7.1.3 关闭应用程序

假设你要关闭运行中的应用程序。比方说，在微服务架构中，你有多个微服务应用的实例运行在云上，其中某个实例有问题了，你决定关闭该实例并让云服务提供商为你重启这个有问题的应用程序。在这个场景中，Actuator的/shutdown端点就很有用了。

为了关闭应用程序，你要往/shutdown发送一个`POST`请求。例如，可以用命令行工具`curl`来关闭应用程序：

```
$ curl -X POST http://localhost:8080/shutdown
```
很显然，关闭运行中的应用程序是件危险的事情，因此这个端点默认是关闭的。如果没有显式地开启这个功能，那么`POST`请求的结果是这样的：
```
{"message":"This endpoint is disabled"}
```  
要开启该端点，可以将`endpoints.shutdown.enabled`设置为`true`。举例来说，可以把如下内容加入application.yml，借此开启/shutdown端点：

```
endpoints:
  shutdown:
    enabled: true
```

打开/shutdown端点后，你要确保并非任何人都能关闭应用程序。这时应该保护/shutdown端点，只有经过授权的用户能关闭应用程序。在7.5节里你将看到如何保护Actuator端点。

### 7.1.4 获取应用信息

Spring Boot Actuator还有一个有用的端点。/info端点能展示各种你希望发布的应用信息。针对该端点的`GET`请求的默认响应是这样的：

```
{}
```
很显然，一个空的JSON对象没什么用。但你可以通过配置带有`info`前缀的属性向/info端点的响应添加内容。例如，你希望在响应中添加联系邮箱。可以在application.yml里设置名为`info.contactEmail`的属性：

```
info:
  contactEmail: support@myreadinglist.com
```

现在再访问/info端点，就能得到如下响应：

```
{
  "contactEmail":"support@myreadinglist.com"
}
```

这里的属性也可以是嵌套的。例如，假设你希望提供联系邮箱和电话。在application.yml里可以配置如下属性：

```
info:
  contact:
    email: support@myreadinglist.com
    phone: 1-888-555-1971
```

/info端点返回的JSON会包含一个`contact`属性，其中有`email`和`phone`属性：

```
{
  "contact":{
    "email":"support@myreadinglist.com",
    "phone":"1-888-555-1971"
  }
}
```

向/info端点添加属性只是定制Actuator行为的众多方式之一。稍后在7.4节里，我们还会看到其他配置与扩展Actuator的方式。但现在，先让我们来看看如何保护Actuator的端点。

| [PREV](https://github.com/5202m/spring-boot-in-action-zh-cn/blob/master/06WallsCh06-6.3.md) | [NEXT](https://github.com/5202m/spring-boot-in-action-zh-cn/blob/master/07WallsCh07-7.2.md) |
