## 5.4 创建可部署的产物

在基于Maven和Gradle的传统Java项目中，构建系统负责产生部署单元——一般是JAR文件或WAR文件。然而，有了Spring Boot CLI，我们可以简单地通过`spring`命令在命令行里运行应用程序。

这是否就意味着要部署一个Spring Boot CLI应用程序，必须在服务器上安装CLI，并手工在命令行里启动应用程序呢？在部署生产环境时，这看起来相当不方便（不用说，这还很危险）。

在第8章里我们会讨论更多部署Spring Boot应用程序的方法。此刻，让我告诉你另一个CLI窍门。针对基于CLI的阅读列表应用程序，在命令行执行如下命令：
```
$ spring jar ReadingList.jar .
```
这会将整个项目打包成一个可执行的JAR文件，包含所有依赖、Groovy和一个嵌入式Tomcat。打包完成后，就可以像下面这样在命令行里运行了（无需CLI）：
```
$ java -jar ReadingList.jar
```
除了可以在命令行里运行外，可执行的JAR文件也能部署到多个平台服务器（Platform as a Service，PaaS）云平台里，包括Pivotal Cloud Foundry和Heroku，在第8章里你会看到相关内容。

## 5.5 小结

Spring Boot CLI利用了Spring Boot自动配置和起步依赖的便利之处，并将其发扬光大。借由Groovy语言的优雅，CLI能让我们在最少的代码噪声下开发Spring应用程序。

本章中我们彻底重写了第2章里的阅读列表应用程序，只是这次我们用Groovy把它写成了Spring Boot CLI应用程序。通过自动添加很多常用包和类的`import`语句，CLI让Groovy更优雅。它还可以自动解析很多依赖库。

对于CLI无法自动解析的库，基于CLI的应用程序可以利用Grape的`@Grab`注解，不用构建说明也能显式地声明依赖。Spring Boot的CLI扩展了`@Grab`注解，针对很多常用库依赖，只需声明Module ID就可以了。

最后，你还了解了如何用Spring Boot CLI来执行测试和构建可部署产物，这些通常都是由构建系统来负责的。

Spring Boot和Groovy结合得很好，两者的简洁性相辅相成。在第6章，我们还会看到Spring Boot和Groovy是如何协同的——Spring Boot是Grails最新版本的核心。

| [PREV](https://github.com/5202m/spring-boot-in-action-zh-cn/blob/master/05WallsCh05-5.4.md) | [NEXT](https://github.com/5202m/spring-boot-in-action-zh-cn/blob/master/06WallsCh06-6.6.md) |
