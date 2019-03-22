# 第5章 Groovy与Spring Boot CLI

>本章内容

>- 自动依赖与`import`
- 获取依赖
- 测试基于CLI的应用程序

有些东西真的很适合在一起：花生酱和果酱，Abbott和Costello***{![两位都是美国的喜剧演员，详见[https://en.wikipedia.org/wiki/Abbott_and_Costello](https://en.wikipedia.org/wiki/Abbott_and_Costello)。——译者注]}***，电闪和雷鸣，牛奶和饼干。每样东西都很棒，但搭配起来就更赞了。

到目前为止，我们已经看到了Spring Boot带来的不少好东西，包括自动配置和起步依赖。要是再搭配上Groovy的优雅，就能起到一加一大于二的效果。

在本章中，我们会了解Spring Boot CLI。这是一个命令行工具，将强大的Spring Boot和Groovy结合到一起，针对Spring应用程序形成了一套简单而又强大的开发工具。为了演示Spring Boot CLI的强大之处，我们会回到第2章的阅读列表应用程序，利用CLI的优势，以Groovy重写这个应用程序。

## 5.1 开发Spring Boot CLI应用程序

大部分针对JVM平台的项目都用Java语言开发，引入了诸如Maven或Gradle这样的构建系统，以生成可部署的产物。实际上，我们在第2章开发的阅读列表应用程序就遵循这套模型。

最近版本的Java语言有不少改进。然而，即便如此，Java还是有一些严格的规则为代码增加了不少噪声。行尾分号、类和方法的修饰符（比如`public`和`private`）、getter和setter方法，还有`import`语句在Java中都有自己的作用，但它们同代码的本质无关，因而造成了干扰。从开发者的角度来看，代码噪声是阻力——编写代码时是阻力，试图阅读代码时更是阻力。如果能消除一部分代码噪声，代码的开发和阅读可以更加方便。

同理，Maven和Gradle这样的构建系统在项目中也有自己的作用，但你还得为此开发和维护构建说明。如果能直接构建，项目也会更加简单。

在使用Spring Boot CLI时，没有构建说明文件。代码本身就是构建说明，提供线索指引CLI解析依赖，并生成用于部署的产物。此外，配合Groovy，Spring Boot CLI提供了一种开发模型，消除了几乎所有代码噪声，带来了畅通无阻的开发体验。

在最简单的情况下，编写基于CLI的应用程序就和编写第1章里的那个Groovy脚本一样简单。不过，要用CLI编写更完整的应用程序，就需要设置一个基本的项目结构来容纳项目代码。我们马上用它重写阅读列表应用程序。

### 5.1.1 设置CLI项目

我们要做的第一件事是创建目录结构，容纳项目。与基于Maven或Gradle的项目不同，Spring Boot CLI项目并没有严格的项目结构要求。实际上，最简单的Spring Boot CLI应用程序就是一个Groovy脚本，可以放在文件系统的任意目录里。对阅读列表应用程序而言，你应该创建一个干净的新目录来存放代码，把它们和你电脑上的其他东西分开。
```
$ mkdir readinglist
```
此处我将目录命名为readinglist，但你可以随意命名。比起找个地方放置代码，名字并不重要。

我们还需要两个额外的目录存放静态Web内容和Thymeleaf模板。在readinglist目录里创建两个新的目录，名为static和templates。
```
$ cd readinglist
$ mkdir static
$ mkdir templates
```
这些目录的名字和基于Java的项目中src/main/resources里的目录同名。虽然Spring Boot并不像Maven和Gradle那样，对目录结构有严格的要求，但Spring Boot会自动配置一个Spring `ResourceHttpRequestHandler`查找static目录（还有其他位置）的静态内容。还会配置Thymeleaf来解析templates目录里的模板。

说到静态内容和Thymeleaf模板，那些文件的内容和我们在第2章里创建的一样。因此你不用担心稍后需要回忆它们，直接把style.css复制到static目录，把readingList.html复制到templates目录。

此时，阅读列表项目的目录结构应该是这样的：

>原书P94，离5.1.2标题最近的代码

现在项目已经设置好了，我们准备好编写Groovy代码了。

### 5.1.2 通过Groovy消除代码噪声

Groovy本身是种优雅的语言。与Java不同，Groovy并不要求有`public`和`private`这样的限定符，也不要求在行尾有分号。此外，归功于Groovy的简化属性语法（GroovyBeans），JavaBean的标准访问方法没有存在的必要了。

随之而来的结果是，用Groovy编写`Book`领域类相当简单。如果在阅读列表项目的根目录里创建一个新的文件，名为Book.groovy，那么在这里编写如下Groovy类。
```
class Book {
    Long id
    String reader
    String isbn
    String title
    String author
    String description
}
```
如你所见，Groovy类与它的Java类相比，大小完全不在一个量级。这里没有setter和getter方法，没有`public`和`private`修饰符，也没有分号。Java中常见的代码噪声不复存在，剩下的内容都在描述书的基本信息。

> __Spring Boot CLI中的JDBC与JPA__

>你也许已经注意到了，`Book`的Groovy实现与第2章里的Java实现有所不同，上面没有添加JPA注解。这是因为这里要用Spring的`JdbcTemplate`，而非Spring Data JPA访问数据库。

> 有好几个不错的理由能解释这个例子为什么选择JDBC而非JPA。首先，在使用Spring的`JdbcTemplate`时，我可以多用几种不同的方法，展示Spring Boot的更多自动配置技巧。选择JDBC的最主要原因是，Spring Data JPA在生成仓库接口的自动实现时要求有一个.class文件。当你在命令行里运行Groovy脚本时，CLI会在内存里编译脚本，并不会产生.class文件。因此，当你在CLI里运行脚本时，Spring Data JPA并不适用。

>但CLI和Spring Data JPA并非完全不兼容。如果使用CLI的`jar`命令把应用程序打包成一个JAR文件，结果文件里就会包含所有Groovy脚本编译后的.class文件。当你想部署一个用CLI开发的应用程序时，在CLI里构建并运行JAR文件是一个不错的选择。但是如果你想在开发时快速看到开发内容的效果，这种做法就没那么方便了。

既然我们定义好了`Book`领域类，就开始编写仓库接口吧。首先，编写`ReadingListRepository`接口（位于ReadingListRepository.groovy）：
```
interface ReadingListRepository {

    List<Book> findByReader(String reader)

    void save(Book book)

}
```

除了没有分号，以及接口上没有`public`修饰符，`ReadingListRepository`的Groovy版本和与之对应的Java版本并无二致。最显著的区别是它没有扩展`JpaRepository`。本章我们不用Spring Data JPA，既然如此，我们就不得不自己实现`ReadingListRepository`。**代码清单5-1**就是JdbcReadingListRepository.groovy的内容。

__代码清单5-1 `ReadingListRepository`的Groovy JDBC实现__

>原书P95~96代码

>文字

>Inject JdbcTemplate=注入`JdbcTemplate`

>RowMapper closure=`RowMapper`闭包

以上代码的大部分内容在实现一个典型的基于`JdbcTemplate`的仓库。它自动注入了一个`JdbcTemplate`对象的引用，用它查询数据库获取图书（在`findByReader()`方法里），将图书保存到数据库（在`save()`方法里）。

因为编写过程采用了Groovy，所以我们在实现中可以使用一些Groovy的语法糖。举个例子，在`findByReader()`里，调用`query()`时可以在需要`RowMapper`实现的地方传入一个Groovy闭包。***{![为了公平对待Java，在Java 8里我们可以用Lambda（和方法引用）做类似的事情。]}***此外，闭包中创建了一个新的`Book`对象，在构造时设置对象的属性。

在考虑数据库持久化时，我们还需要创建一个名为schema.sql的文件。其中包含创建`Book`表所需的SQL。仓库在发起查询时依赖这个数据表：
```
create table Book (
        id identity,
        reader varchar(20) not null,
        isbn varchar(10) not null,
        title varchar(50) not null,
        author varchar(50) not null,
        description varchar(2000) not null
);
```

稍后我会解释如何使用schema.sql。现在你只需要知道，把它放在Classpath的根目录（即项目的根目录），就能创建出查询用的`Book`表了。

Groovy的所有部分差不多都齐全了，但还有一个Groovy类必须要写。这样Groovy化的阅读列表应用程序才完整。我们需要编写一个`ReadingListController`的Groovy实现来处理Web请求，为浏览器提供阅读列表。在项目的根目录，要创建一个名为ReadingListController.groovy的文件，内容如代码清单5-2所示。

__代码清单5-2 处理展示和添加Web请求的`ReadingListController`__

>原书P97代码

>文字：

>Inject ReadingListRepository：注入`ReadingListRepository`

>Fetch reading list：获取阅读列表

>Populate model：设置模型

>Return view name：返回视图名称

>Save book：保存图书

>Redirect after POST：`POST`后重定向

这个`ReadingListController`和第2章里的版本有很多相似之处。主要的不同在于，Groovy的语法消除了类和方法的修饰符、分号、访问方法和其他不必要的代码噪声。

你还会注意到，两个处理器方法都用`def`而非`String`来定义。两者都没有显式的`return`语句。如果你喜欢在方法上说明类型，喜欢显式的`retrun`语句，加上就好了——Groovy并不在意这些细节。

在运行应用程序之前，还要做一件事。那就是创建一个新文件，名为Grabs.groovy，内容包括如下三行：

```
@Grab("h2")
@Grab("spring-boot-starter-thymeleaf")
class Grabs {}
```

稍后我们再来讨论这个类的作用，现在你只需要知道类上的`@Grab`注解会告诉Groovy在启动应用程序时自动获取一些依赖的库。

不管你信还是不信，我们已经可以运行这个应用程序了。我们创建了一个项目目录，向其中复制了一个样式表和Thymeleaf模板，填充了一些Groovy代码。接下来，用Spring Boot CLI（在项目目录里）运行即可：

```
$ spring run .
```

几秒后，应用程序完全启动。打开浏览器，访问[http://localhost:8080](https://localhost:8080)。如果一切正常，你应该就能看到和第2章一样的阅读列表应用程序。

成功啦！只用了几页纸的篇幅，你就写出了简单而又完整的Spring应用程序！

此时此刻你也许会好奇这是怎么办到的。

- *没有Spring配置*，Bean是如何创建并组装的？`JdbcTemplate` Bean又是从哪来的？
- *没有构建文件*，Spring MVC和Thymeleaf这样的依赖库是哪来的？
- *没有`import`语句*。如果不通过`import`语句来指定具体的包，Groovy如何解析`JdbcTemplate`和`RequestMapping`的类型？
- *没有部署应用*，Web服务器从何而来？

实际上，我们编写的代码看起来不止缺少分号。这些代码究竟是怎么运行起来的？

### 5.1.3 发生了什么

你可能已经猜到了，Spring Boot CLI在这里不仅仅是便捷地使用Groovy编写了Spring应用程序。Spring Boot CLI施展了很多技能。

* CLI可以利用Spring Boot的自动配置和起步依赖。
* CLI可以检测到正在使用的特定类，自动解析合适的依赖库来支持那些类。
* CLI知道多数常用类都在哪些包里，如果用到了这些类，它会把那些包加入Groovy的默认包里。
* 应用自动依赖解析和自动配置后，CLI可以检测到当前运行的是一个Web应用程序，并自动引入嵌入式Web容器（默认是Tomcat）供应用程序使用。

仔细想想，这些才是CLI提供的最重要的特性。Groovy语法只是额外的福利！

通过Spring Boot CLI运行阅读列表应用程序，表面看似平凡无奇，实则大有乾坤。CLI尝试用内嵌的Groovy编译器来编译Groovy代码。虽然你不知道，但实际上，未知类型（比如`JdbcTemplate`、`Controller`及`RequestMapping`，等等）最终会使代码编译失败。

但CLI不会放弃，它知道只要把Spring Boot JDBC起步依赖加入Classpath就能找到`JdbcTemplate`。它还知道把Spring Boot的Web起步依赖加入Classpath就能找到Spring MVC的相关类。因此，CLI会从Maven仓库（默认为Maven中心仓库）里获取那些依赖。

如果此时CLI重新编译，那还是会失败，因为缺少`import`语句。但CLI知道很多常用类的包。利用定制Groovy编译器默认包导入的功能之后，CLI把所有需要用到的包都加入了Groovy编译器的默认导入列表。

现在CLI可以尝试再一次编译了。假设没有其他CLI能力范围外的问题（比如，存在CLI不知道的语法或类型错误），代码就能完成编译。CLI将通过内置的启动方法（与基于Java的例子里的`main()`方法类似）运行应用程序。

此时，Spring Boot自动配置就能发挥作用了。它发现Classpath里存在Spring MVC（因为CLI解析了Web起步依赖），就自动配置了合适的Bean来支持Spring MVC，还有嵌入式Tomcat Bean供应用程序使用。它还发现Classpath里有`JdbcTemplate`，所以自动创建了`JdbcTemplate` Bean，注入了同样自动创建的`DataSource` Bean。

说起`DataSource` Bean，这只是Spring Boot自动配置创建的众多Bean中的一个。Spring Boot还自动配置了很多Bean来支持Spring MVC中的Thymeleaf模板。正是由于我们使用`@Grab`注解向Classpath里添加了H2和Thymeleaf，这才触发了针对嵌入式H2数据库和Thymeleaf的自动配置。

`@Grab`注解的作用是方便添加CLI无法自动解析的依赖。虽然它看上去很简单，但实际上这个小小的注解作用远比你想象得要大。让我们仔细看看这个注解，看看Spring Boot CLI是如何通过一个Artifact名称找到这么多常用依赖，看看整个依赖解析的过程是如何配置的。
