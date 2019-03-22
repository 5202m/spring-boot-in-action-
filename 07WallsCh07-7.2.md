## 7.2 连接Actuator的远程shell

Actuator通过REST端点提供了不少非常有用的信息。另一个深入运行中应用程序内部的方式是使用远程shell。Spring Boot集成了CRaSH，一种能嵌入任意Java应用程序的shell。Spring Boot还扩展了CRaSH，添加了不少Spring Boot特有的命令，提供了与Actuator端点类似的功能。

要使用远程shell，只需加入远程shell的起步依赖即可。你需要这样的Gradle依赖：
```
compile("org.springframework.boot:spring-boot-starter-remote-shell")
```
如果用Maven构建项目，你需要在pom.xml文件里添加如下依赖：

```
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-remote-shell</artifactId>
</dependency>
```

如果要用Spring Boot CLI来运行你所开发的应用程序，则需要如下`@Grab`注解：

```
@Grab("spring-boot-starter-remote-shell")
```

添加了远程shell依赖后，就可以构建并运行应用程序了。在启动的时候，可以看到要写进日志的一行密码。这行密码所在的行大概是这样的：

```
Using default security password: efe30c70-5bf0-43b1-9d50-c7a02dda7d79
```

与这个密码搭配使用的用户名是user。密码本身是随机生成的，每次运行应用程序时都会有所变化。

现在你可以通过SSH工具连接shell了，它监听的端口号是2000。如果你用的是Unix的`ssh`命令，那么它看起来大概是这样的：

> P142 代码

太棒了！你已经连上shell了。现在应该做什么？

远程shell提供了24个可以在运行应用程序上下文中执行的命令，其中大部分都是CRaSH自带的。但Spring Boot也添加了一些。表7-4列出了这些Spring Boot特有的命令。

__表7-4 Spring Boot提供的CRaSH shell命令__

| 命令 | 描述 |
|-----|-----|
| `autoconfig` | 生成自动配置说明报告，和/autoconfig端点输出的内容类似，只是把JSON换成了纯文本 |
| `beans` | 列出Spring应用程序上下文里的Bean，与/beans端点输出的内容类似 |
| `endpoint` | 调用Actuator端点|
| `metrics` | 显示Spring Boot的度量信息，与/metrics端点类似，但显示的是实时更新的数据 |

让我们看看如何使用这些Spring Boot添加的shell命令。

### 7.2.1 查看`autoconfig`报告

`autoconfig`命令生成了一个与Actuatord的/autoconfig端点类似的报告。图7-1是`autoconfig`命令输出的内容截屏片段。

>P143 ![Image text](https://raw.githubusercontent.com/5202m/spring-boot-in-action-zh-cn/master/imgs/figure-7.1.png)

__图7-1 `autoconfig`命令的输出__

如你所见，结果分为两组——匹配和不匹配，和/autoconfig端点的结果一样。实际上，唯一的显著区别在于，`autoconfig`命令输出的是文本，而/autoconfig端点输出的是JSON，其他都一样。

我不打算去讲CRaSH自己提供的shell命令，但你可能想把`autoconfig`命令的输出和CRaSH的`less`命令用管道串起来：

```
> autoconfig | less
```

`less`命令和Unix shell里的同名命令很相似，能让你穿梭于文件中。`autoconfig`的输出很长，但`less`命令会让它更容易读取和查阅。

### 7.2.2 列出应用程序的Bean

`autoconfig`shell命令的输出和/autoconfig端点的输出类似，但也有不同。对比之下，你会发现`beans`命令的输出和/beans端点的输出一样。截屏如图7-2所示。

>P144 ![Image text](https://raw.githubusercontent.com/5202m/spring-boot-in-action-zh-cn/master/imgs/figure-7.2.png)

__图7-2 `beans`命令的输出__

和/beans端点一样，`beans`命令会以JSON格式列出Spring应用程序上下文里所有的Bean，包括所依赖的Bean。

### 7.2.3 查看应用程序的度量信息

`metrics`shell命令会输出与Actuator的/metrics端点一样的信息。/metrics端点以JSON格式输出当前度量信息的快照，而`metrics`命令则会接管shell，以实时仪表盘显示结果。图7-3就是`metrics`命令的仪表盘。

>P144 ![Image text](https://raw.githubusercontent.com/5202m/spring-boot-in-action-zh-cn/master/imgs/figure-7.3.png)

__图7-3 `metrics`命令的仪表盘__

`metrics`命令的实时仪表盘很难在书里以静态图片演示。但你可以试着想象一下，内存、堆、线程在不断消耗和释放，随着类的加载，仪表盘里显示的数量也会随之变化，以反映当前值。

一旦看完了`metrics`命令提供的度量信息，按Ctrl+C就能回到shell了。

### 7.2.4 调用Actuator端点

你现在应该已经意识到了，并非所有的Actuator端点都有对应的shell命令。这是否意味着shell不能完全代替Actuator端点呢？是否仍要直接查询这些端点来获取Actuator提供的内部信息呢？虽然shell没能完全匹配上这些端点，但`endpoint`命令可以让你在shell里调用Actuator的端点。

首先，你要知道自己想调用哪个端点。在shell提示符中键入`endpoint list`就能获得端点的列表，如图7-4所示。请注意，列表中的端点用的是它们的Bean名称，而非URL路径。

>P 144 ![Image text](https://raw.githubusercontent.com/5202m/spring-boot-in-action-zh-cn/master/imgs/figure-7.4.png)

__图7-4 获得端点列表__

如果想在shell里调用其中某个端点，你可以使用`endpoint invoke`命令，传入不带Endpoint后缀的Bean名称。举例来说，要调用健康检查端点，可以在shell提示符里键入`endpoint invoke health`，如图7-5所示。

>P146 ![Image text](https://raw.githubusercontent.com/5202m/spring-boot-in-action-zh-cn/master/imgs/figure-7.5.png)

__图7-5 调用健康检查端点__

请注意，这些端点返回的信息都是原始格式的，即未格式化过的JSON文档。虽然在shell里调用Actuator的端点不错，但输出结果很难阅读。就这个问题，自带的功能帮不上忙。但如果爱折腾，你也可以创建一个自定义的CRaSH shell命令，通过管道接受未格式化的JSON，然后美化输出。你总是可以剪切黏贴`endpoint`命令的输出，将其放入你喜欢的工具进行阅读或格式化。

| [PREV](https://github.com/5202m/spring-boot-in-action-zh-cn/blob/master/07WallsCh07-7.1.md) | [NEXT](https://github.com/5202m/spring-boot-in-action-zh-cn/blob/master/07WallsCh07-7.3.md) |
