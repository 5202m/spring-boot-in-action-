## 7.4 定制Actuator

虽然Actuator提供了很多运行中Spring Boot应用程序的内部工作细节，但难免和你的需求有所偏差。也许你并不需要它提供的所有功能，想要关闭一些也说不定。或者，你需要对Actuator稍作扩展，增加一些自定义的度量信息，以满足你对应用程序的需求。

实际上，Actuator有多种定制方式，包括以下五项

* 重命名端点。
* 启用和禁用端点。
* 自定义度量信息。
* 创建自定义仓库来存储跟踪数据。
* 插入自定义的健康指示器。

接下来，我们会了解如何定制Actuator，让它满足我们的需要。先来看一个最简单的定制：重命名Actuator端点。

### 7.4.1 修改端点ID

每个Actuator端点都有一个ID用来决定端点的路径，比方说，/beans端点的默认ID就是`beans`。

如果端点的路径是由ID决定的，那么可以通过修改ID来改变端点的路径。你要做的就是设置一个属性，属性名是`endpoints.endpoint-id.id`。

我们用/shutdown端点来做个演示，它会响应发往/shutdown的`POST`请求。假设你想让它处理发往/kill的`POST`请求，可以通过如下YAML为/shutdown赋予一个新的ID，也就是新的路径：

```
endpoints:
  shutdown:
    id: kill
```

重命名端点、修改其路径的理由很多。最明显的理由就是，端点的命名要和团队的术语保持一致。你也可能想重命名端点，让那些熟悉默认名称的人找不到它，借此增加一些安全感。

遗憾的是，重命名端点并不能真的起到保护作用，顶多是让黑客慢点找到它们。我们会在7.5节看到如何保护这些Actuator端点。现在先让我们来看看如何禁用某个（或全部）不希望别人访问的端点。

### 7.4.2 启用和禁用端点

虽然Actuator的端点都很有用，但你不一定需要全部这些端点。默认情况下，所有端点（除了/shutdown）都启用。我们已经看过如何设置`endpoints.shutdown.enabled`为`true`，以此开启/shutdown端点（详见7.1.1节）。用同样的方式，你可以禁用其他的端点，将`endpoints.endpoint-id.enabled `设置为`false`。

例如，要禁用/metrics端点，你要做的就是将`endpoints.metrics.enabled`属性设置为`false`。在application.yml里做如下设置：
```
endpoints:
  metrics:
    enabled: false
```
如果你只想打开一两个端点，那就先禁用全部端点，然后启用那几个你要的，这样更方便。例如，考虑如下application.yml片段：
```
endpoints:
  enabled: false
  metrics:
    enabled: true
```
正如以上片段所示，`endpoints.enabled`设置为`false`就能禁用Actuator的全部端点，然后将`endpoints.metrics.enabled`设置为`true`重新启用/metrics端点。

### 7.4.3 添加自定义度量信息

在7.1.2节中，你看到了如何从/metrics端点获得运行中应用程序的内部度量信息，包括内存、垃圾回收和线程信息。这些都是非常有用且信息量很大的度量值，但你可能还想定义自己的度量，用来捕获应用程序中的特定信息。

比方说，我们想要知道用户往阅读列表里保存了多少次图书，最简单的方法就是在每次调用`ReadingListController`的`addToReadingList()`方法时增加计数器值。计数器很容易实现，但这个不断变化的总计值如何同/metrics端点发布的度量信息一起发布出来呢？

再假设我们想要获得最后保存图书的时间戳。时间戳可以通过调用`System.currentTimeMillis()`来获取，但如何在/metrics端点里报告该时间戳呢？

实际上，自动配置允许Actuator创建`CounterService`的实例，并将其注册为Spring的应用程序上下文中的Bean。`CounterService`这个接口里定义了三个方法，分别用来增加、减少或重置特定名称的度量值，代码如下：

```
package org.springframework.boot.actuate.metrics;

public interface CounterService {
  void increment(String metricName);
  void decrement(String metricName);
  void reset(String metricName);
}
```
Actuator的自动配置还会配置一个`GaugeService`类型的Bean。该接口与`CounterService`类似，能将某个值记录到特定名称的度量值里。`GaugeService`看起来是这样的：

```
package org.springframework.boot.actuate.metrics;

public interface GaugeService {
  void submit(String metricName, double value);
}
```

你无需实现这些接口。Spring Boot已经提供了两者的实现。我们所要做的就是把它们的实例注入所需的Bean，在适当的时候调用其中的方法，更新想要的度量值。

针对上文提到的需求，我们需要把`CounterService`和`GaugeService` Bean注入`ReadingListController`，然后在`addToReadingList()`方法里调用其中的方法。代码清单7-9是`ReadingListController`里的相关变动：

__代码清单7-9 使用注入的`CounterService`和`GaugeService`__

```
@Controller
@RequestMapping("/")
@ConfigurationProperties("amazon")
public class ReadingListController {
  ...
  private CounterService counterService;

  @Autowired
  public ReadingListController(
      ReadingListRepository readingListRepository,
      AmazonProperties amazonProperties,
      CounterService counterService,
      GaugeService gaugeService) {
    this.readingListRepository = readingListRepository;
    this.amazonProperties = amazonProperties;
    this.counterService = counterService;
    this.gaugeService = gaugeService;
  }

  ...

  @RequestMapping(method=RequestMethod.POST)
  public String addToReadingList(Reader reader, Book book) {
    book.setReader(reader);
    readingListRepository.save(book);
    counterService.increment("books.saved");
    gaugeService.submit(
      "books.last.saved", System.currentTimeMillis());
    return "redirect:/";
  }
}
```
>文字

>Inject the counter and gauge services：注入`CounterService`和`GaugeService`

>Increment “books.saved”：增加`books.saved`的值

>Record “books.last.saved”：记录`books.last.saved`的值

修改后的`ReadingListController`使用了自动织入机制，通过控制器的构造方法注入`CounterService`和`GaugeService`，随后把它们保存在实例变量里。此后，`addToReadingList()`方法每次处理请求时都会调用`counterService.increment("books.saved")`和`gaugeService.submit("books.last.saved")`来调整度量值。

尽管`CounterService`和`GaugeService`用起来很简单，但还是有一些度量值很难通过增加计数器或记录指标值来捕获。对于那些情况，我们可以实现`PublicMetrics`接口，提供自己需要的度量信息。该接口定义了一个`metrics()`方法，返回一个`Metric`对象的集合：

```
package org.springframework.boot.actuate.endpoint;

public interface PublicMetrics {
  Collection<Metric<?>> metrics();
}
```

为了解`PublicMetrics`的使用方法，这里假设我们想报告一些源自Spring应用程序上下文的度量值——应用程序上下文启动的时间、Bean及Bean定义的数量，这些都包含进来会很有意思。顺便再报告一下添加了`@Controller`注解的Bean的数量。代码清单7-10给出了相应`PublicMetrics`实现的代码。

__代码清单7-10 发布自定义度量信息__

```
package readinglist;
import java.util.ArrayList;
import java.util.Collection;
import java.util.List;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.actuate.endpoint.PublicMetrics;
import org.springframework.boot.actuate.metrics.Metric;
import org.springframework.context.ApplicationContext;
import org.springframework.stereotype.Component;
import org.springframework.stereotype.Controller;

@Component
public class ApplicationContextMetrics implements PublicMetrics {
  private ApplicationContext context;

  @Autowired
  public ApplicationContextMetrics(ApplicationContext context) {
    this.context = context;
  }

  @Override
  public Collection<Metric<?>> metrics() {
    List<Metric<?>> metrics = new ArrayList<Metric<?>>();
    metrics.add(new Metric<Long>("spring.context.startup-date",
        context.getStartupDate()));

    metrics.add(new Metric<Integer>("spring.beans.definitions",
        context.getBeanDefinitionCount()));

    metrics.add(new Metric<Integer>("spring.beans",
        context.getBeanNamesForType(Object.class).length));

    metrics.add(new Metric<Integer>("spring.controllers",
        context.getBeanNamesForAnnotation(Controller.class).length));

    return metrics;
  }
}
```
> 文字

>Record startup date：记录启动时间

>Record bean definition count：记录Bean定义数量

>Record bean count：记录Bean数量

>Record controller bean count：记录控制器类型的Bean数量

Actuator会调用`metrics()`方法，收集`ApplicationContextMetrics`提供的度量信息。该方法调用了所注入的`ApplicationContext`上的方法，获取我们想要报告为度量的数量。每个度量值都会创建一个`Metrics`实例，指定度量的名称和值，将其加入要返回的列表。

创建`ApplicationContextMetrics`，并在`ReadingListController`里使用`CounterService`和`GaugeService`之后，我们可以在/metrics端点的响应中找到如下条目：

```
{
  ...
  spring.context.startup-date: 1429398980443,
  spring.beans.definitions: 261,
  spring.beans: 272,
  spring.controllers: 2,
  books.count: 1,
  gauge.books.save.time: 1429399793260,
  ...
}
```

当然，这些度量的实际值会根据添加了多少书、何时启动应用程序及何时保存最后一本书而发生变化。在这个例子里，你一定会好奇为什么`spring.controllers`是2。因为这里算上了`ReadingListController`以及Spring Boot提供的`BasicErrorController`。

### 7.4.4 创建自定义跟踪仓库

默认情况下，/trace端点报告的跟踪信息都存储在内存仓库里，100个条目封顶。一旦仓库满了，就开始移除老的条目，给新的条目腾出空间。在开发阶段这没什么问题，但在生产环境中，大流量会造成跟踪信息还没来得及看就被丢弃。

为了避免这个问题，你可以声明自己的`InMemoryTraceRepository` Bean，将它的容量调整至100以上。如下配置类可以将容量调整至1000个条目：

```
package readinglist;
import org.springframework.boot.actuate.trace.InMemoryTraceRepository;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class ActuatorConfig {
  @Bean
  public InMemoryTraceRepository traceRepository() {
    InMemoryTraceRepository traceRepo = new InMemoryTraceRepository();
    traceRepo.setCapacity(1000);
    return traceRepo;
  }
}
```

仓库容量翻了十倍，跟踪信息的保存时间应该会更久。不过，繁忙到一定程度，应用程序还是可能在你查看这些信息前将其丢弃。这是一个内存存储的仓库，还要避免容量增长太多，影响应用程序的内存使用。

除了上述方法，我们还可以将那些跟踪条目存储在其他地方——既不消耗内存，又能长久保存的地方。只需实现Spring Boot的`TraceRepository`接口即可：

```
package org.springframework.boot.actuate.trace;
import java.util.List;
import java.util.Map;

public interface TraceRepository {
  List<Trace> findAll();
  void add(Map<String, Object> traceInfo);
}
```

如你所见，`TraceRepository`只要求我们实现两个方法：一个方法查找所有存储的`Trace`对象，另一个保存了一个`Trace`，包含跟踪信息的`Map`对象。

作为演示，假设我们创建了一个使用MongoDB数据库存储跟踪信息的`TraceRepository`实例。代码清单7-11演示了如何实现这个`TraceRepository`。

__代码清单7-11 往MongoDB保存跟踪数据__

```
package readinglist;
import java.util.Date;
import java.util.List;
import java.util.Map;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.actuate.trace.Trace;
import org.springframework.boot.actuate.trace.TraceRepository;
import org.springframework.data.mongodb.core.MongoOperations;
import org.springframework.stereotype.Service;

@Service
public class MongoTraceRepository implements TraceRepository {

  private MongoOperations mongoOps;

  @Autowired
  public MongoTraceRepository(MongoOperations mongoOps) {
    this.mongoOps = mongoOps;
  }

  @Override
  public List<Trace> findAll() {
    return mongoOps.findAll(Trace.class);
  }

  @Override
  public void add(Map<String, Object> traceInfo) {
    mongoOps.save(new Trace(new Date(), traceInfo));
  }
}
```
> 文字

> Inject MongoOperations：注入`MongoOperations`

> Fetch all trace entries：获取所有跟踪条目

> Save a trace entry：保存一个跟踪条目

`findAll()`方法很直白，用注入的`MongoOperations`来查找全部`Trace`对象。`add()`方法稍微有趣一点，用当前时间和含有跟踪信息的`Map`创建了一个`Trace`对象，然后通过`MongoOperations.save()`将其保存下来。唯一的问题是，`MongoOperations`是哪里来的？

为了使用`MongoTraceRepository`，我们需要保证Spring应用程序上下文里先有一个`MongoOperations` Bean。得益于Spring Boot的起步依赖和自动配置，做到这一点只需添加MongoDB起步依赖即可。你需要如下Gradle依赖：

```
compile("org.springframework.boot:spring-boot-starter-data-mongodb")
```

如果你用的是Maven，则需要如下依赖：

```
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
```

添加了这个起步依赖后，Spring Data MongoDB和所依赖的库会添加到应用程序的Classpath里。Spring Boot会自动配置所需的Bean，以便使用MongoDB数据库。这些Bean里就包括`MongoOperations`。另外，你需要确保和`MongoOperations`通讯的MongoDB服务器正常运行。

### 7.4.5 插入自定义健康指示器

如前文所述，Actuator自带了很多健康指示器，能满足常见需求，比如报告应用程序使用的数据库和消息代理的健康情况。但如果你的应用程序需要和一些没有健康指示器的系统交互，那该怎么办呢？

我们的阅读列表里有指向Amazon的图书链接，可以报告一下Amazon是否可以访问。当然，Amazon不太可能宕机，但不怕一万就怕万一，所以让我们为Amazon创建一个健康指示器吧。代码清单7-12演示了相关`HealthIndicator`的实现。

__代码清单7-12 自定义一个Amazon健康指示器__

```
package readinglist;
import org.springframework.boot.actuate.health.Health;
import org.springframework.boot.actuate.health.HealthIndicator;
import org.springframework.stereotype.Component;
import org.springframework.web.client.RestTemplate;

@Component
public class AmazonHealth implements HealthIndicator {

  @Override
  public Health health() {
    try {
      RestTemplate rest = new RestTemplate();
      rest.getForObject("http://www.amazon.com", String.class);
      return Health.up().build();
    } catch (Exception e) {
      return Health.down().build();
    }
  }

}
```
>文字

>Send request to Amazon：向Amazon发送请求

>Report “down” health：报告`DOWN`状态

`AmazonHealth`类并没有什么花哨的地方。`health()`方法只是使用Spring的`RestTemplate`向Amazon首页发起了一个`GET`请求。如果请求成功，则返回一个表明Amazon状态为`UP`的`Health`对象。如果请求发生异常，则`health()`返回一个标明Amazon状态为`DOWN`的`Health`对象。

下面是/health端点响应的一个片段。这里可以看出，如果Amazon不可访问，你会看到什么。

```
{
  "amazonHealth": {
    "status": "DOWN"
  },
  ...
}
```

你不会相信我等Amazon宕机等了多久，就为了能看到上面的结果！***{![实际上我并没有等太久。我只是把电脑的网络断开了。没有网就没有Amazon。]}***

除了简单的状态之外，如果你还想向健康记录里添加其他附加信息，可以调用`Health`构造器的`withDetail()`方法。例如，要添加异常消息，将其作为健康记录的`reason`字段，可以让`catch`块返回这样一个`Health`对象：

```
return Health.down().withDetail("reason", e.getMessage()).build();
```

修改后，当Amazon无法访问时，健康记录看起来是这样的：

```
"amazonHealth": {
  "reason": "I/O error on GET request for
             \"http://www.amazon.com\":www.amazon.com;
             nested exception is java.net.UnknownHostException:
             www.amazon.com",
  "status": "DOWN"
},
```

如果你有很多附加信息，可以多次调用`withDetail()`方法，每次设置一个想要放入健康记录的附加字段。
