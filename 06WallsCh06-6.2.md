## 6.2 使用Groovy Server Pages定义视图

到目前为止，我们都在用Thymeleaf模板定义阅读列表应用程序的视图。除了Thymeleaf，Spring Boot还支持Freemarker、Velocity和基于Groovy的模板。无论选择哪种模板，你要做的就是添加合适的起步依赖，在Classpath根部的templates/目录里编写模板。自动配置会处理剩下的事情。

Grails项目也提供GSP的自动配置。如果你想在Spring Boot应用程序里使用GSP，必须向项目里添加Spring Boot的GSP库：
```
compile("org.grails:grails-gsp-spring-boot:1.0.0")
```
和Spring Boot提供的其他视图模板一样，库放在Classpath里就会触发自动配置，设置所需的视图解析器，以便在Spring MVC的视图层里使用GSP。

剩下的就是为应用程序编写GSP模板了。在阅读列表应用程序中，我们要把Thymeleaf的readingList.html文件用GSP的形式重写，放在readingList.gsp文件（位于src/main/resources/templates）里。代码清单6-5就是新的GSP模板的代码。

__代码清单6-5 GSP编写的阅读列表应用程序主视图__

>原书P113~114代码

>文字

>List the books：罗列图书

>The book form：图书表单

>The CSRF token： CSRF令牌

如你所见，GSP模板中使用了表达式语言引用（用`${}`包围的部分）以及GSP标签库（例如`<g:if>`和`<g:each>`）。这并不是Thymeleaf那样的纯HTML。但如果习惯用JSP，你会很熟悉这种方式，而且会觉得这是一个不错的选择。

代码里的绝大部分内容和第2章、第3章的Thymeleaf模板类似，映射GSP模板上的元素。但是有一点要注意，你必须要放一个隐藏域，其中包含CSRF（Cross-Site Request Forgery）令牌。Spring Security在提交`POST`请求时要求带有这个令牌，Thymeleaf在呈现HTML时会自动包含这个令牌，但在GSP里你必须在隐藏域显式地包含它。

图6-1是GSP模板的显示效果，其中添加了一些图书。

虽然GORM和GSP这样的Grails特性很吸引人，让Spring Boot应用程序更简单，但你在这里还不能真正体验Grails。让我们再往Spring Boot的花生酱里放一点Grails巧克力。现在让我们来看看Grails 3如何将两者合二为一，带来完整的Spring Boot和Grails开发体验。

>P115 ![Image text](https://raw.githubusercontent.com/5202m/spring-boot-in-action-zh-cn/master/imgs/figure-6.1.png)

__图6-1 使用了GSP模板的阅读列表__
