## 3.3 定制应用程序错误页面

错误总是会发生的，那些在生产环境里最健壮的应用程序偶尔也会遇到麻烦。虽然减小用户遇到错误的概率很重要，但让应用程序展现一个好的错误页面也同样重要。

近年来，富有创意的错误页已经成为了一种艺术。如果你曾见到过GitHub.com的星球大战错误页，或者是DropBox.com的Escher立方体错误页的话，你就能明白我在说什么了。

我不知道你在使用阅读列表应用程序时有没有碰到错误，如果有的话，你看到的页面应该和图3-1里的很像。

>P72 ![Image text](https://raw.githubusercontent.com/5202m/spring-boot-in-action-zh-cn/master/imgs/figure-3.1.png)

__图3-1 Spring Boot的默认白标错误页面__

Spring Boot默认提供这个“白标”（whitelabel）错误页，这是自动配置的一部分。虽然这比错误栈跟踪要好一点，但和网上那些伟大的错误页艺术品却不可同日而语。为了让你的应用程序故障页变成大师级作品，你需要为应用程序创建一个自定义的错误页。

Spring Boot自动配置的默认错误处理器会查找名为error的视图，如果找不到就用默认的白标错误视图，如图3-1所示。因此，最简单的方法就是创建一个自定义视图，让解析出的视图名为error。

这一点归根到底取决于错误视图解析时的视图解析器，包括：

- 实现了Spring的`View`接口的Bean，其 ID为`error`（由Spring的`BeanNameViewResolver`所解析）
- 如果配置了Thymeleaf，则是名为error.html的Thymeleaf模板
- 如果配置了FreeMarker，则是名为error.ftl的FreeMarker模板
- 如果配置了Velocity，则是名为error.vm的Velocity模板
- 如果是用JSP视图，则是名为error.jsp的JSP模板

因为我们的阅读列表应用程序使用了Thymeleaf，所以我们要做的就是创建一个名为error.html的文件，把它和其他的应用程序模板一起放在模板文件夹里。代码清单3-7是一个简单有效的错误页，可以用来代替默认的白标错误页。

__代码清单3-7 阅读列表应用程序的自定义错误页__

>原书P73

>**文字**

>Show requested path：显示请求路径

>Show error details：显示错误明细

这个自定义的错误模板应该命名为error.html，放在模板目录里，这样Thymeleaf模板解析器才能找到它。在典型的Maven或Gradle项目里，这就意味着要把该文件放在src/main/resources/templates中，运行时它就在Classpath的根目录里。

基本上，这个简单的Thymeleaf模板就是显示一张图片和一些提示错误的文字。其中有两处特别的信息需要呈现：错误的请求路径和异常消息。但这还不是错误页上的全部细节。默认情况下，Spring Boot会为错误视图提供如下错误属性。

- `timestamp`：错误发生的时间。
- `status`：HTTP状态码。
- `error`：错误原因。
- `exception`：异常的类名。
- `message`：异常消息（如果这个错误是由异常引起的）。
- `errors`：`BindingResult`异常里的各种错误（如果这个错误是由异常引起的）。
- `trace`：异常栈跟踪（如果这个错误是由异常引起的）。
- `path`：错误发生时请求的URL路径。

其中某些属性，比如`path`，在向用户交待问题时还是很有用的。其他的，比如`trace`，用起来要保守一点，将其隐藏，或者是用得聪明点，让错误页尽可能对用户友好。

请注意，模板里还引用了一张名为MissingPage.png的图片。图片的实际内容并不重要，所以尽情挑选适合你的图片就好了，但请一定将它放在src/main/resources/static或src/main/resources/public里，这样应用程序运行时才能找到它。

图3-2是发生错误时用户会看到的页面。虽然它算不上一件艺术品，但还是把应用程序错误页的艺术水准稍微提高了那么一点。

>-P74 ![Image text](https://raw.githubusercontent.com/5202m/spring-boot-in-action-zh-cn/master/imgs/figure-3.2.png)

__图3-2 遇到错误时展现的自定义错误页__

## 3.4 小结

Spring Boot消除了Spring应用程序中经常要用到的很多样板式配置。让Spring Boot处理全部配置，你可以仰仗它来配置那些适合你的应用程序的组件。当自动配置无法满足需求时，Spring Boot允许你覆盖并微调它提供的配置。

覆盖自动配置其实很简单，就是显式地编写那些没有Spring Boot时你要做的Spring配置。Spring Boot的自动配置被设计为优先使用应用程序提供的配置，然后才轮到自己的自动配置。

即使自动配置合适，你仍然需要调整一些细节。Spring Boot会开启多个属性解析器，让你通过环境变量、属性文件、YAML文件等多种方式来设置属性，以此微调配置。这套基于属性的配置模型也能用于应用程序自己定义的组件，可以从外部配置源加载属性并注入到Bean里。

Spring Boot还自动配置了一个简单的白标错误页，虽然它比异常和栈跟踪要更友好一点，但在艺术性方面还有很大的提升空间。幸运的是，Spring Boot提供了好几种选项来自定义或完全替换这个白标错误页，以满足应用程序的特定风格。

现在我们已经用Spring Boot写了一个完整的应用程序，我们会验证它能否满足预期。除了自己在浏览器里手工点点之外，我们还应该要写一些自动化、可重复运行的测试来检查这个应用程序，证明它能正确运作。这也是我们在第4章里要做的事。
