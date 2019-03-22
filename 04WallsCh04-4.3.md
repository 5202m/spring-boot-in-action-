## 4.3 测试运行中的应用程序

说到测试Web应用程序，我们还没接触实质内容。在真实的服务器里启动应用程序，用真实的Web浏览器访问它，这样比使用模拟的测试引擎更能展现应用程序在用户端的行为。

但是，用真实的Web浏览器在真实的服务器上运行测试会很麻烦。虽然构建时的插件能把应用程序部署到Tomcat或者Jetty里，但它们配置起来多有不便。而且测试这么多，几乎不可能隔离运行，也很难不启动构建工具。

然而Spring Boot找到了解决方案。它支持将Tomcat或Jetty这样的嵌入式Servlet容器作为运行中的应用程序的一部分，可以运用相同的机制，在测试过程中用嵌入式Servlet容器来启动应用程序。

Spring Boot的`@WebIntegrationTest`注解就是这么做的。在测试类上添加`@WebIntegrationTest`注解，可以声明你不仅希望Spring Boot为测试创建应用程序上下文，还要启动一个嵌入式的Servlet容器。一旦应用程序运行在嵌入式容器里，你就可以发起真实的HTTP请求，断言结果了。

举例来说，考虑一下代码清单4-5里的那段简单的Web测试。这里采用`@WebIntegrationTest`，在服务器里启动了应用程序，以Spring的`RestTemplate`对应用程序发起HTTP请求。

__代码清单4-5 测试运行在服务器里的Web应用程序__

>原书P86代码

>**文字**

>Runs test in server：在服务器里运行测试

>Performs GET request：发起`GET`请求

>Asserts HTTP 404 (not found) response：判断HTTP 404 (not found)响应

虽然这个测试非常简单，但足以演示如何使用`@WebIntegrationTest`在服务器里启动应用程序。要判断实际启动的服务器究竟是哪个，可以遵循在命令行里运行应用程序时的逻辑。默认情况下，会有一个监听8080端口的Tomcat启动。但是，如果Classpath里有的话，Jetty或者Undertow也能启动这些服务器。

测试方法的主体部分假设应用程序已经运行，监听了8080端口。它使用了Spring的`RestTemplate`对一个不存在的页面发起请求，判断服务器的响应是否为HTTP 404 (not found)。如果返回了其他响应，则测试失败。

### 4.3.1 用随机端口启动服务器

前面提到过，此处的默认行为是启动服务器监听8080端口。在一台机器上一次只运行一个测试的话，这没什么问题，因为没有其他服务器监听8080端口。但如果你和我一样，本机总是有其他服务器在监听8080端口，那该怎么办？这时测试会失败，因为端口冲突，服务器启动不了。一定要有更好的办法才行。

幸运的是，让Spring Boot在随机选择的端口上启动服务器很方便。一种办法是将`server.port`属性设置为`0`，让Spring Boot选择一个随机的可用端口。`@WebIntegrationTest`的`value`属性接受一个`String`数组，数组中的每项都是键值对，形如`name=value`，用来设置测试中使用的属性。要设置`server.port`，你可以这样做：
```
@WebIntegrationTest(value={"server.port=0"})
```
另外，因为只要设置一个属性，所以还能有更简单的形式：
```
@WebIntegrationTest("server.port=0")
```
通过`value`属性来设置属性通常还算方便。但`@WebIntegrationTest`还提供了一个`randomPort`属性，更明确地表示让服务器在随机端口上启动。你可以将`randomPort`设置为`true`，启用随机端口：
```
@WebIntegrationTest(randomPort=true)
```
既然我们在随机端口上启动了服务器，就需要在发起Web请求时确保使用正确的端口。此时的`getForObject()`方法在URL里硬编码了8080端口。如果端口是随机选择的，那在构造请求时又该怎么确定正确的端口呢？

首先，我们需要以实例变量的形式注入选中的端口。为了方便，Spring Boot将`local.server.port`的值设置为了选中的端口。我们只需使用Spring的`@Value`注解将其注入即可：
```
@Value("${local.server.port}")
private int port;
```
有了端口之后，只需对`getForObject()`稍作修改，使用这个`port`就好了：
```
rest.getForObject(
    "http://localhost:{port}/bogusPage", String.class, port);
```
这里我们在URL里把硬编码的8080改为`{port}`占位符。在`getForObject()`调用里把`port`属性作为最后一个参数传入，就能确保该占位符被替换为注入`port`的值了。

### 4.3.2 使用Selenium测试HTML页面

`RestTemplate`对于简单的请求而言使用方便，是测试REST端点的理想工具。但是，就算它能对返回HTML页面的URL发起请求，也不方便对页面内容或者页面上执行的操作进行断言。结果HTML里的内容最好能够精确判断（这种测试很脆弱）。不过你无法轻易判断页面上选中的内容，或者执行诸如点击链接或提交表单这样的操作。

对于HTML应用程序测试，有一个更好的选择——Selenium（[www.seleniumhq.org](http://www.seleniumhq.org)），它的功能远不止提交请求和获取结果。它能实际打开一个Web浏览器，在浏览器的上下文中执行测试。Selenium尽量接近手动执行测试，但与手工测试不同。Selenium的测试是自动的，而且可以重复运行。

为了用Selenium测试阅读列表应用程序，让我们先写一个测试来获取首页，为新书填写表单，提交表单，随后判断返回的页面里是否包含新添加的图书。

首先需要把Selenium作为测试依赖添加到项目里：
```
testCompile("org.seleniumhq.selenium:selenium-java:2.45.0")
```
现在就可以编写测试了。代码清单4-6是一个基本的Selenium测试模板，使用了Spring Boot的`@WebIntegrationTest`。

__代码清单4-6 在Spring Boot里使用Selenium测试的模板__

>原书P88~89代码

>**文字**

>Starts on a random port：用随机端口启动

>Injects the port：注入端口号

>Sets up Firefox driver：配置Firefox驱动

>Shuts down browser：关闭浏览器

和之前更简单的Web测试一样，这个类添加了`@WebIntegrationTest`注解，将`randomPort`设置为`true`，这样应用程序启动后会运行一个监听随机端口的服务器。同样，端口号注入`port`属性，这样我们就能用它来构造指向运行中应用程序的URL了。

静态方法`openBrowser()`会创建一个`FirefoxDriver`的实例，它将打开Firefox浏览器（需要在运行测试的服务器上安装该浏览器）。我们的测试方法将通过`FirefoxDriver`实例来执行浏览器操作。在页面上查找元素时，`FirefoxDriver`配置了10秒的等候时间（以防元素加载过慢）。

测试执行完毕，我们需要关闭Firefox浏览器。因此要在`closeBrowser()`里要调用`FirefoxDriver`实例的`quit()`方法，关闭浏览器。

>__选择浏览器__

> 虽然我们用Firefox进行了测试，但Selenium还提供了不少其他浏览器的驱动，包括IE、Google的Chrome，还有Apple的Safari。测试可以使用其他浏览器。你也可以使用你想支持的各种浏览器，这也许也是个不错的想法。

现在可以开始编写测试方法了，给你提个醒，我们想要加载首页，填充并发送表单，然后判断登录的页面是否包含刚刚添加的新书。代码清单4-7演示了如何用Selenium实现这个功能。

__代码清单4-7 用Selenium测试阅读列表应用程序__

>原书P89~90代码

>**文字**

>Fetches the home page：获取主页

>Asserts an empty book list：判断图书列表是否为空

>Fills in and submits form：填充并发送表单

>Asserts new book in list：判断列表中是否包含新书

该测试方法所做的第一件事是使用`FirefoxDriver`来发起`GET`请求，获取阅读列表的主页，随后查找页面里的一个`<div>`元素，从它的文本里判断列表里没有图书。

接下来的几行查找表单里的元素，使用驱动的`sendKeys()`方法模拟敲击键盘事件（实际上就是用给定的值填充那些表单域）。最后，找到`<form>`元素并提交。

提交的表单经处理后，浏览器就会跳到一个页面，上面的列表包含了新添加的图书。因此最后几行查找列表里的`<dt>`和`<dd>`元素，判断其中是否包含测试表单里提交的数据。

运行测试时，你会看到浏览器打开，加载阅读列表应用程序。如果够仔细，你还会看到填充表单的过程，就好像幽灵在操作，当然，并没有幽灵使用你的应用程序——这只是一个测试。

这个测试里最值得注意的是，`@WebIntegrationTest`可以为我们启动应用程序和服务器，这样Selenium才可以用Web浏览器执行测试。但真正有趣的是你可以使用IDE的测试功能来运行测试，随便你想跑几次都行，无需依赖构建过程中的某些插件启动服务器。

要是你觉得使用Selenium进行测试很实用，可以阅读Yujun Liang和Alex Collins的*Selenium WebDriver in Practice*（[http://manning.com/liang/](http://manning.com/liang/)），该书更深入地讨论了Selenium测试的细节。

## 4.4 小结

测试是开发高质量软件的重要一环。没有好的测试，你永远无法保证应用程序能像你期望的那样运行。

单元测试专注于单一组件或组件中的一个方法，此处并不一定要使用Spring。Spring提供了一些优势和技术——松耦合、依赖注入和接口驱动设计。这些都简化了单元测试的编写。但Spring不用直接涉足单元测试。

集成测试会涉及众多组件，这时就需要Spring帮忙了。实际上，如果Spring在运行时负责拼装那些组件，那么Spring在集成测试里同样应该肩负这一职责。

Spring Framework以JUnit类运行器的方式提供了集成测试支持，JUnit类运行器会加载Spring应用程序上下文，把上下文里的Bean注入测试。Spring Boot在Spring的集成测试之上又增加了配置加载器，以Spring Boot的方式加载应用程序上下文，包括了对外置属性的支持和Spring Boot日志。

Spring Boot还支持容器内测试Web应用程序，让你能用和生产环境一样的容器启动应用程序。这样一来，测试在验证应用程序行为的时候，会更加接近真实的运行环境。

此时我们已经构建了一个相当完整的应用程序（虽然有点简单），它利用Spring Boot的起步依赖和自动配置来处理低级工作，让我们专心开发应用程序。我们也看到了如何使用Spring Boot的支持来测试应用程序。在后续几章里，我们会看到一些不同的东西，了解让Spring Boot应用程序开发更加简单的Groovy。在第5章，我们会先了解Grails框架的一些特性，看看它们在Spring Boot中的用途。
