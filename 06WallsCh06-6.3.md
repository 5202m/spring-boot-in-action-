## 6.3 结合Spring Boot与Grails 3

Grails一直都是构建于Spring、Groovy、Hibernate和其他巨人肩膀之上的高阶框架。到了Grails 3，Grails已经基于Spring Boot，带来了令人愉悦的开发体验。Grails开发者和Spring Boot开发者都能驾轻就熟。

要使用Grails 3，首先要进行安装。在Mac OS X和大部分Unix系统上，最简单的安装方法是在命令行里使用SDKMAN：
```
$ sdk install grails
```
如果你用的是Windows，或者无法使用SDKMAN，就需要下载二进制发布包。解压后要将bin目录添加到系统路径里去。

无论用哪种安装方式，你都可以在命令行中查看Grails的版本，验证安装是否成功：
```
$ grails -version
```
如果安装成功，现在就可以创建Grails项目了。

### 6.3.1 创建新的Grails项目

在Grails项目中，你会使用`grails`命令行工具执行很多任务，包括创建项目。要创建阅读列表项目，可以这样使用`grails`命令：
```
$ grails create-app readinglist
```
正如这个命令的名字所示，`create-app`命令创建了一个新的应用程序项目。在这个例子里，项目名是readinglist。

等`grails`工具创建完应用程序，`cd`到了`readinglist`目录里，看看所创建的内容吧。图6-2应该就是你看到的项目结构的概览。

在这个项目目录结构里，你应该认出了一些熟悉的东西。这里有一个Gradle的构建说明文件和配置（build.gradle和gradle.properties）。src目录里还有一个标准的Gradle项目结构，但是grails-app应该是里面最有趣的目录。如果用过老版本的Grails，你就会知道这个目录的作用。这里面放的是你写的控制器、领域类和其他构成Grails项目的代码。

>P116 ![Image text](https://raw.githubusercontent.com/5202m/spring-boot-in-action-zh-cn/master/imgs/figure-6.2.png)

__图6-2 Grails 3项目的目录结构__

如果再深挖一下，打开build.gradle文件，会发现一些更熟悉的东西。首先，构建说明文件里使用了Spring Boot的Gradle插件：
```
apply plugin: "spring-boot"
```
这意味着你能像使用其他Spring Boot应用程序那样构建并运行这个Grails应用程序。

你还应该注意到，依赖里有不少有用的Spring Boot库：

```
dependencies {
  compile 'org.springframework.boot:spring-boot-starter-logging'
  compile("org.springframework.boot:spring-boot-starter-actuator")
  compile "org.springframework.boot:spring-boot-autoconfigure"
  compile "org.springframework.boot:spring-boot-starter-tomcat"
  ...
}
```
这些库为Grails应用程序提供了Spring Boot的自动配置、日志，还有Actuator及嵌入式Tomcat。把应用当作可执行JAR运行时，这个Tomcat可以提供服务。

实际上，这是一个Spring Boot项目，同时也是Grails项目，因为Grails 3就是构建在Spring Boot的基础上的。

#### 运行应用程序

运行Grails应用程序最直接的方式是在命令行里使用`grails`工具的`run-app`命令：
```
$ grails run-app
```
就算一行代码都还没写，我们也能运行应用程序，在浏览器里进行访问。一旦应用程序启动，就可以在浏览器里访问[http://localhost:8080](http://localhost:8080)。你应该能看到类似图6-3的页面。

>P117 ![Image text](https://raw.githubusercontent.com/5202m/spring-boot-in-action-zh-cn/master/imgs/figure-6.3.png)

__图6-3 全新的Grails应用程序__

在Grails里运行应用程序要使用`run-app`命令，这种方式已经用了很多年，上个版本的Grails也是这样。Grails 3项目的Gradle说明里使用了Spring Boot的Gradle插件，你还可以用各种运行Spring Boot项目的方式来运行这个应用程序。此处通过Gradle引入了`bootRun`任务：
```
$ gradle bootRun
```
你还可以构建项目，运行生成的可执行JAR文件：
```
$ gradle build
...
$ java -jar build/lib/readingList-0.1.jar
```
当然，构建产生的WAR文件可以部署到你喜欢的各种Servlet 3.0容器里。

在开发早期就能运行应用程序，这一点十分方便，能帮你确认应用程序已正确初始化。但是这时应用程序还没做什么有意思的事情，在初始化后的项目上做什么完全取决于我们。接下来，开始定义领域模型吧。

### 6.3.2 定义领域模型

阅读列表应用程序里的核心领域模型是`Book`类。虽然我们可以手工创建Book.groovy文件，但通常还是用`grails`工具来创建领域模型类比较好。因为它知道该把文件放到哪里，并且能在同一时间生成各种相关内容。

要创建`Book`类，我们会使用`grails`工具的`create-domain-class`命令：
```
$ grails create-domain-class Book
```
这条命令会生成两个源文件：一个Book.groovy文件和一个BookSpec.groovy文件。后者是一个Spock说明，用来测试`Book`类。一开始这个文件是空的，你可以填入各种测试内容来验证`Book`的各种功能。

Book.groovy文件里定义了`Book`类，你可以在grails-app/domain/readingList里找到这个文件。它一开始基本没什么内容：
```
package readinglist
class Book {

  static constraints = {
  }
}
```
我们需要添加一些字段来定义一本书，比如书名、作者和ISBN。在添加了这些字段后，Book.groovy看起来是这样的：
```
package readinglist
class Book {
  static constraints = {
  }

  String reader
  String isbn
  String title
  String author
  String description

}
```
静态的`constraints`变量里可以定义各种附加在`Book`实例上的验证约束。本章中，我们主要关注阅读列表应用程序的构建，看看如何基于Spring Boot构建应用程序，不会太关注验证的问题。因此，这里的`constraints`内容为空。当然，如果有需要的话，你可以随意添加约束。可以参考一下*Grails in Action  Second Edition *（Manning，2014），作者是Glen Smith和Peter Ledbrook。***{![虽然*Grails in Action Second Edition*主讲Grails 2，但你在Grails 2里了解到的大部分内容都适用于Grails 3。]}***

为了使用Grails，我们要保持阅读列表应用程序的简洁，要和第2章的程序一致。因此，接下来我们要创建`Reader`领域模型，还有控制器。

### 6.3.3 开发Grails控制器

有了领域模型，通过`grails`工具创建出控制器就很容易了。关于控制器，有几个命令可供选择。

* `create-controller`：创建空控制器，让开发者来编写控制器的功能。
* `generate-controller`：生成一个控制器，其中包含特定领域模型类的基本CRUD操作。
* `generate-all`：生成针对特定领域模型类的基本CRUD控制器，及其视图。

脚手架控制器很好用，也是Grails中比较知名的特性，但我们仍然会保持简洁，写一个仅包含必要功能的控制器，能匹配第2章里的应用程序功能就好。因此，我们用`create-controller`命令来创建原始的控制器，然后填入所需的方法：
```
$ grails create-controller ReadingList
```
这个命令会在grails-app/controllers/readingList里创建一个名为`ReadingListController`的控制器：
```
package readinglist
class ReadingListController {

  def index() { }
}
```
一行代码都不用改，这个控制器就能运行了，虽然它干不成什么事。此时，它能处理发往/readingList的请求，将请求转给grails-app/views/readingList/index.gsp里定义的视图（现在还没有，我们稍后会创建的）。

我们需要控制器来显示图书列表，还有添加新书的表单。我们还需要提交表单，将新书保存到数据库里的功能。下面的代码就是我们所需要的`ReadingListController`。

__代码清单6-6 改写`ReadingListController`__

>原书120代码

>文字

>Fetch books into model：获取图书填充到模型里

>Save the book：保存图书

虽然相比等效的Java控制器，代码长度大大缩短，但这个版本的`ReadingListController`功能已经基本完整。它可以处理发往/readingList的`GET`请求，获取并展示图书列表。在表单提交后，它还会处理`POST`请求，保存图书，随后重定向回index动作（由`index()`方法来处理）。

太不可思议了，我们已经基本完成了Grails版本的阅读列表应用程序。剩下的就是创建一个视图，显示图书列表和表单。

### 6.3.4 创建视图

Grails应用程序通常都用GSP模板来做视图。你已经看到过如何在Spring Boot应用程序里使用GSP了，因此，此处的模板并不会和6.2节里的模板有太多不同。

我们要做的是，利用Grails提供的布局设施，将公共的设计风格运用到整个应用程序里。如代码清单6-7所示，这就是个很简单的修改。

__代码清单6.7 一个适用于Grails的GSP模板，包含布局__

```
<!DOCTYPE html>
<html>
  <head>
    <meta name="layout" content="main"/>
    <title>Reading List</title>
    <link rel="stylesheet"
          href="/assets/main.css?compile=false"  />
    <link rel="stylesheet"
          href="/assets/mobile.css?compile=false"  />
    <link rel="stylesheet"
          href="/assets/application.css?compile=false"  />
  </head>

  <body>
    <h2>Your Reading List</h2>

    <g:if test="${bookList && !bookList.isEmpty()}">
      <g:each in="${bookList}" var="book">
      <dl>
        <dt class="bookHeadline">
          ${book.title}</span> by ${book.author}
          (ISBN: ${book.isbn}")
        </dt>
        <dd class="bookDescription">
          <g:if test="${book.description}">
          ${book.description}
          </g:if>
          <g:else>
          No description available
          </g:else>
        </dd>
      </dl>
      </g:each>
    </g:if>
    <g:else>
      <p>You have no books in your book list</p>
    </g:else>

    <hr/>

    <h3>Add a book</h3>
    <g:form action="save">
    <fieldset class="form">
      <label for="title">Title:</label>
      <g:field type="text" name="title" value="${book?.title}"/><br/>
      <label for="author">Author:</label>
      <g:field type="text" name="author"
                          value="${book?.author}"/><br/>
      <label for="isbn">ISBN:</label>
      <g:field type="text" name="isbn" value="${book?.isbn}"/><br/>
      <label for="description">Description:</label><br/>
      <g:textArea name="description" value="${book?.description}"
                                     rows="5" cols="80"/>
    </fieldset>
    <fieldset class="buttons">
      <g:submitButton name="create" class="save"
        value="${message(code: 'default.button.create.label',
                                              default: 'Create')}" />
    </fieldset>
    </g:form>

  </body>
</html>
```

>文字

>Use the main layout：使用了`main`布局

>List the books：列出图书

>The book form：图书表单

在`<head>`元素里我们移除了引用样式表的`<link>`标签。这里放了一个`<meta>`标签，引入了Grails应用程序的`main`布局。这样一来，应用程序就能用上Grails的外观了，运行的效果如图6-4所示。

>P122 ![Image text](https://raw.githubusercontent.com/5202m/spring-boot-in-action-zh-cn/master/imgs/figure-6.4.png)

__图6-4 应用通用Grails风格的阅读列表应用程序__

虽然Grails风格比之前用的简单的样式表更吸引眼球。但很显然的是，要让阅读列表应用程序更好看，还有一些工作要做。首先要让应用程序和Grails不那么像，和我们的想象更接近一点。修改应用程序的样式表在本书的讨论范围之外，但如果你对样式微调感兴趣，可以在grails-app/assets/stylesheets目录里找到样式表文件。

## 6.4 小结

Grails和Spring Boot都旨在让开发者的生活更简单，大大简化基于Spring的开发模型，因此两者看起来是互相竞争的框架。但在本章中，我们看到了两者如何结合在一起，综合优势。

我们了解了如何向典型的Spring Boot应用程序中添加GORM和GSP视图，这两个都是知名的Grails特性。GORM是Spring Boot里一个很受欢迎的特性，能让你直接针对领域模型执行持久化操作，消除了对模型仓库的需求。

随后我们认识了Grails 3，即Grails构建于Spring Boot之上的最新版本。在开发Grails 3应用程序时，你也在使用Spring Boot，可以使用Spring Boot的全部特性，包括自动配置。

在本章和第5章里，我们看到了如何结合Groovy和Spring Boot，消除Java语言无法避免的那些代码噪声。

在第7章，我们要将关注点从开发Spring Boot应用程序转移到Spring Boot Actuator上，看看它如何帮助我们了解应用程序的运行情况。
