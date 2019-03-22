## 7.3 通过JMX监控应用程序

除了REST端点和远程shell，Actuator还把它的端点以MBean的方式发布了出来，可以通过JMX来查看和管理。JMX是种不错的Spring Boot应用程序管理方式，如果你已在用JMX管理应用程序中的其他MBean，则尤其如此。

Actuator的端点都发布在org.springframework.boot域下。比如，你想要查看应用程序的请求映射关系，那么可以看一下图7-6（通过JConsole查看请求映射端点）。

>P146 ![Image text](https://raw.githubusercontent.com/5202m/spring-boot-in-action-zh-cn/master/imgs/figure-7.6.png)

__图7-6 通过JConsole查看请求映射端点__

如你所见，在`requestMappingEndpoint`下可以找到请求映射端点，位于org.springframework.boot域中的`Endpoint`下。`Data`属性中包含了该端点所要输出的JSON内容。

和其他MBean一样，端点MBean有可供调用的操作。大部分端点MBean只有访问操作，返回其中的某个属性，但关闭端点提供了一些有趣（同时具有毁灭性）的操作，如图7-7所示。

>P147 ![Image text](https://raw.githubusercontent.com/5202m/spring-boot-in-action-zh-cn/master/imgs/figure-7.7.png)

__图7-7 调用该端点的关闭按钮__

如果你想要关闭应用程序（或者喜欢冒险的生活），那么关闭应用的端点正合你意。如图7-7所示，这个界面就等你点击shutdown按钮调用该端点。请小心，这里没有“后悔药”，也没有“你确定吗？”之类的提示。

接下来你会看图7-8。

>P147 ![Image text](https://raw.githubusercontent.com/5202m/spring-boot-in-action-zh-cn/master/imgs/figure-7.8.png)

__图7-8 应用程序立马关闭__

在那以后，你的应用程序就关闭了。应用已经关闭，自然就没办法发布其他用来重启它的MBean操作。你必须重启，和一开始的启动方式一样。
