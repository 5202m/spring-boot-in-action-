# 附录A Spring Boot开发者工具

Spring Boot 1.3引入了一组新的开发者工具，可以让你在开发时更方便地使用Spring Boot，包括如下功能。

* *自动重启*：当Classpath里的文件发生变化时，自动重启运行中的应用程序。
* *LiveReload支持*：对资源的修改自动触发浏览器刷新。
* *远程开发*：远程部署时支持自动重启和LiveReload。
* *默认的开发时属性值*：为一些属性提供有意义的默认开发时属性值。

Spring Boot的开发者工具采取了库的形式，可以作为依赖加入项目。如果你使用Gradle来构建项目，可以像下面这样在build.gradle文件里添加开发工具：

```
compile "org.springframework.boot:spring-boot-devtools"
```

在Maven POM里添加`<dependency>`是这样的：

```
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-devtools</artifactId>
</dependency>
```

当应用程序以完整打包好的JAR或WAR文件形式运行时，开发者工具会被禁用，所以没有必要在构建生产部署包前移除这个依赖。

## A.1 自动重启

在激活了开发者工具后，Classpath里对文件做任何修改都会触发应用程序重启。为了让重启速度够快，不会修改的类（比如第三方JAR文件里的类）都加载到了基础类加载器里，而应用程序的代码则会加载到一个单独的重启类加载器里。检测到变更时，只有重启类加载器重启。

有些Classpath里的资源变更后不需要重启应用程序。像Thymeleaf这样的视图模板可以直接编辑，不用重启应用程序。在/static或/public里的静态资源也不用重启应用程序，所以Spring Boot开发者工具会在重启时排除掉如下目录：/META-INF/resources、/resources、/static、/public和/templates。

可以设置`spring.devtools.restart.exclude`属性来覆盖默认的重启排除目录。例如，你只排除/static和/templates目录，可以像这样设置`spring.devtools.restart.exclude`：

```
spring:
  devtools:
    restart:
      exclude: /static/**,/templates/**
```

另一方面，如果你想彻底关闭自动重启，可以将`spring.devtools.restart.enabled`设置为`false`：

```
spring:
  devtools:
    restart:
      enabled: false
```

另外，还可以设置一个触发文件，必须修改这个文件才能触发重启。例如，在修改名为.trigger的文件前你都不希望执行重启，那么你只需像这样设置`spring.devtools.restart.trigger-file`属性：

```
spring:
  devtools:
    restart:
      trigger-file: .trigger
```

如果你的IDE会连续编译修改的文件，那触发文件还是很有用的。没有触发文件的话，每次变更都会触发重启。有触发文件，就能保证只有你想重启时才会发生重启（修改触发文件即可）。
