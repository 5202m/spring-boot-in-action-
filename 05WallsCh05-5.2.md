## 5.2 获取依赖

在Spring MVC和`JdbcTemplate`的例子中，为了获取必要的依赖并添加到Classpath里，Groovy编译触发了Spring Boot CLI。这是错误的。但如果需要一个依赖，而没有失败代码来触发自动依赖解析，又或者所需的依赖CLI不知道，那该怎么办？

在阅读列表应用程序中，我们需要Thymeleaf库，这样才能编写使用了Thymeleaf模板的视图。我们还需要H2的库，这样才能拥有嵌入式的H2数据库。但因为没有Groovy代码会直接引用Thymeleaf或H2的类，所以不会有编译错误来触发自动依赖解析。因此，我们要帮一帮CLI，在`Grabs`类上添加`@Grab`依赖。

> __该把`@Grab`注解放在哪里？__

> 并不需要像我们这样，严格将`@Grab`注解放在一个单独的类上。它们在`ReadingListController`或`JdbcReadingListRepository`同样有效。不过，为了便于组织管理，最好创建一个空类，把所有`@Grab`注解放在一起。这样方便在一个地方看到所有显式声明的依赖。

`@Grab`注解源自Groovy Grape（Groovy Adaptable Packaging Engine或Groovy Advanced Packaging Engine）工具。从本质上来说，Grape允许Groovy脚本在运行时下载依赖，无需Maven或Gradle这样的构建工具介入。除了支持`@Grab`注解，Spring Boot CLI还用Grape来获取代码中推断出的依赖。

使用`@Grab`就和描述依赖一样简单。举例来说，假设你想往项目里添加H2数据库，可以往项目的一个Groovy脚本添加如下`@Grab`注解：
```
@Grab(group="com.h2database", module="h2", version="1.4.190")
```
这样能明确地声明依赖的组、模块和版本号。或者，你也可以用更简洁的冒号分割表示依赖，这和Gradle构建说明里的表示方式类似。
```
@Grab("com.h2database:h2:1.4.185")
```
这是两个教科书式的例子，但Spring Boot CLI对`@Grab`做了几处扩展，用起来更简单。

很多依赖不再要求指定版本号了。可以通过下面的方式，用`@Grab`添加H2数据库依赖：
```
@Grab("com.h2database:h2")
```
确切的版本号是由你所使用的CLI的版本来决定的。如果用的是Spring Boot CLI 1.3.0.RELEASE，那么H2依赖的版本会解析为1.4.190。

这还不算完，很多常用依赖还可以省去Group ID，直接在`@Grab`里写上模块的ID。正是这个特性让上文的`@Grab`注解成功加载了H2。
```
@Grab("h2")
```
那你该如何获知某个依赖是需要Group ID和版本号，还是只需要Module ID呢？我在附录D中提供了一个完整的列表，包含了Spring Boot CLI知道的全部依赖。通常，你可以先试一下只写Module ID，如果这样不行，再加上Group ID和版本号。

只用Module ID来表示依赖会很方便，但如果你并不认可Spring Boot选择的版本号怎么办？如果Spring Boot的起步依赖传递引入了一个库的某个版本，但你想要使用修正了bug的新版本又该如何呢？

### 5.2.1 覆盖默认依赖版本

Spring Boot引入了新的`@GrabMetadata`注解，可以和`@Grab`搭配使用，用属性文件里的内容来覆盖默认的依赖版本。

要用`@GrabMetadata`，可以把它加到某个Groovy脚本文件里，提供相应的属性文件来覆盖依赖元数据：
```
@GrabMetadata(“com.myorg:custom-versions:1.0.0”)
```  
这会从Maven仓库的com/myorg目录里加载一个名为custom-versions.properties的文件。文件里的每一行都应该有Group ID和Module ID。以这两个东西为键名，属性则是值。例如，要把H2的默认版本覆盖为1.4.186，可以把`@GrabMetadata`指向一个包含如下内容的属性文件：
```
com.h2database:h2=1.4.186
```
> __使用Spring IO平台__

> 你可能希望让`@GrabMetadata`使用Spring IO平台（[http://platform.spring.io/platform/](http://platform.spring.io/platform/)）上定义的依赖版本。该平台提供了一套依赖和版本。明确哪个版本的Spring能和其他库的什么版本搭配使用。Spring IO平台提供的依赖和版本是Spring Boot已知依赖库的一个超集，包含了很多Spring应用程序经常用到的第三方库。

>   如果你想在Spring IO平台上构建Spring Boot CLI应用程序，只需要在Groovy脚本中添加如下`@GrabMetadata`即可。
>```
@GrabMetadata('io.spring.platform:platform-versions:1.0.4.RELEASE')
```
>  这会覆盖CLI的默认依赖版本，使Spring IO平台定义的版本取而代之。

你可能会有疑问，Grape又是从哪里获取所有这些依赖的呢？这是可配置的吗？让我们来看看你该如何管理Grape获取依赖的仓库集。

### 5.2.2 添加依赖仓库

默认情况下，`@Grab`声明的依赖是从Maven中心仓库（[http://repo1.maven.org/maven2/](http://repo1.maven.org/maven2/)）拉取的。此外，Spring Boot还注册了Spring的里程碑及快照仓库，以便获取Spring项目的预发布版本依赖。对很多项目而言，这就足够了。但要是你的项目需要的库不在这两者之中该怎么办呢？或者你的工作环境在公司防火墙内，必须使用内部仓库又该如何？

没有问题。`@GrabResolver`注解可以让你指定额外的仓库，用来获取依赖。

举个例子，假设你想使用最新的Hibernate，而最新的Hibernate版本只能从JBoss的仓库里获取到。那么你需要通过`@GrabResolver`来添加仓库：
```
@GrabResolver(name='jboss', root=
  'https://repository.jboss.org/nexus/content/groups/public-jboss')
```
这里通过`name`属性将该解析器命名为jboss，通过`root`属性来指定仓库的URL。

你已经了解了Spring Boot CLI是如何编译代码以及自动按需解析已知依赖库的。在`@Grab`的支持下，CLI可以解析各种它无法自动解析的依赖。基于CLI的应用程序无需Maven或Gradle构建说明文件（传统方式开发的Java应用程序需要这个文件）。但解析依赖和编译代码并不是构建过程的全部，项目的构建通常还要执行自动化测试，要是没有构建说明文件，又该如何运行测试呢？
