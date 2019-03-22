# 第2章 开发第一个应用程序

> 本章内容

>- 使用Spring Boot起步依赖
- 自动进行Spring配置

你上次在超市或大型零售商店自己推开门是什么时候？大多数大型商店都安装了带感应功能的自动门，虽然所有门都能让你进入建筑物内，但自动门不用你动手推拉。

与之类似，很多公共场所的卫生间里都装有自动感应水龙头和自动感应纸巾机。虽然没有超市自动门这么普及，但这些设施同样对你没有太多要求，可以很方便地出水和纸巾。

说实话，我已经不记得上次看到制冰盒是什么时候了，更不记得自己往里面倒水制冰或者取冰的事了。我的冰箱就是这么神奇，总是有冰，让我随时都能喝上冰水。

我敢打赌你也能想出无数例子，证明设备让现代生活更加自动化，而不是增加障碍。有了这些自动化的便利设施，你会认为在开发任务里也会出现更多的自动化。但是很奇怪，事实并非如此。

直到最近，要用Spring创建应用程序，你还需要为框架做很多事情。当然，Spring提供了很多优秀的特性，用于开发令人惊讶的应用程序。但是，你需要自己往项目的构建说明文件里添加各种库依赖；你还要自己写配置文件，告诉Spring做什么。

Spring Boot将Spring开发的自动化程度提升到了一个新的高度，在本章我们会看到两种新方法：起步依赖和自动配置。在项目中启用Spring不仅枯燥乏味，还让人分神，你将看到这些基础的Spring Boot特性是如何将你解放出来，让你集中精力开发应用程序的。与此同时，你会写一个很小的Spring应用程序，麻雀虽小，五脏俱全，其中会用上Spring Boot。

## 2.1 运用Spring Boot

你正在阅读本书，说明你是一位读书人。也许你是一个书虫，博览群书；也许你只读自己需要的东西，拿起本书只是为了知道怎么用Spring开发应用程序。

无论何种情况，你都是一位读书人，是读书人便有心维护一个阅读列表，里面是自己想读或者需要读的书。就算没有白纸黑字的列表，至少在你心里会有这么一个列表。***{![如果你不是一个读书人，就把书换成想看的电影、想去的餐厅，只要合适自己就好。]}***

在本书中，我们会构建一个简单的阅读列表应用程序。在这个程序里，用户可以输入想读的图书信息，查看列表，删除已经读过的书。我们将使用Spring Boot来辅助快速开发，各种繁文缛节越少越好。

开始前，我们需要先初始化一个项目。在第1章里，我们看到了好几种从Spring Initializr开始Spring Boot开发的方法。因为选择哪种方法都行，所以要选个最合适的，着手用Spring Boot开发就好了。

从技术角度来看，我们要用Spring MVC来处理Web请求，用Thymeleaf来定义Web视图，用Spring Data JPA来把阅读列表持久化到数据库里，姑且先用嵌入式的H2数据库。虽然也可以用Groovy，但是我们还是先用Java来开发这个应用程序吧。此外，我们使用Gradle作为构建工具。

无论是用Web界面、Spring Tool Suite还是IntelliJ IDEA，只要用了Initializr，你就要确保勾选了Web、Thymeleaf和JPA这几个复选框。还要记得勾上H2复选框，这样才能在开发应用程序时使用这个内嵌式数据库。

至于项目元信息，就随便你写了。以阅读列表为例，我创建项目时使用了图2-1中的信息。

>P25 ![Image text](https://raw.githubusercontent.com/5202m/spring-boot-in-action-zh-cn/master/imgs/figure-2.1.png)

__图2-1 通过Initializr的Web界面初始化阅读列表应用程序__

如果你创建项目时用的是Spring Tool Suite或者IntelliJ IDEA，那么把图2-1的内容适配成IDE需要的东西就好了。

另一方面，如果用Spring Boot CLI来初始化应用程序，可以在命令行里键入以下内容：
```
$ spring init -dweb,data-jpa,h2,thymeleaf --build gradle readinglist
```
请记住，CLI的`init`命令是不能指定项目根包名和项目名的。包名默认是demo，项目名默认是Demo。在项目创建完毕之后，你可以打开项目，把包名demo改为readinglist，把DemoApplication.java改名为ReadingListApplication.java。

项目创建完毕后，你应该能看到一个类似图2-2的项目结构。

>P26 ![Image text](https://raw.githubusercontent.com/5202m/spring-boot-in-action-zh-cn/master/imgs/figure-2.2.png)

__图2-2 初始化后的readinglist项目结构__

这个项目结构基本上和第1章里Initializr生成的结构是一样的，只不过你现在真的要去开发应用程序了，所以让我们先放慢脚步，仔细看看初始化的项目里都有什么东西。

### 2.1.1 查看初始化的Spring Boot新项目

图2-2中值得注意的第一件事是，整个项目结构遵循传统Maven或Gradle项目的布局，即主要应用程序代码位于src/main/java目录里，资源都在src/main/resources目录里，测试代码则在src/test/java目录里。此刻还没有测试资源，但如果有的话，要放在src/test/resources里。

再进一步，你会看到项目里还有不少文件。

- build.gradle：Gradle构建说明文件。
- `ReadingListApplication.java`：应用程序的启动引导类（bootstrap class），也是主要的Spring配置类。
- `application.properties`：用于配置应用程序和Spring Boot的属性。
- `ReadingListApplicationTests.java`：一个基本的集成测试类。

因为构建说明文件里有很多Spring Boot的优点尚未揭秘，所以我打算把最好的留到最后，先让我们来看看`ReadingListApplication.java`。

#### 1.启动引导Spring

`ReadingListApplication`在Spring Boot应用程序里有两个作用：配置和启动引导。首先，这是主要的Spring配置类。虽然Spring Boot的自动配置免除了很多Spring配置，但你还需要进行少量配置来启用自动配置。正如代码清单2-1所示，这里只有一行配置代码。

__代码清单2-1 `ReadingListApplication.java`不仅是启动引导类，还是配置类__

>原书P27代码

>**文字**

>Enable component-scanning and auto-configuration-开启组件扫描和自动配置

>Bootstrap the applicantion-负责启动引导应用程序

`@SpringBootApplication`开启了Spring的组件扫描和Spring Boot的自动配置功能。实际上，`@SpringBootApplication`将三个有用的注解组合在了一起。

- Spring的`@Configuration`：标明该类使用Spring基于Java的配置。虽然本书不会写太多配置，但我们会更倾向于使用基于Java而不是XML的配置。
- Spring的`@ComponentScan`：启用组件扫描，这样你写的Web控制器类和其他组件才能被自动发现并注册为Spring应用程序上下文里的Bean。本章稍后会写一个简单的Spring MVC控制器，使用`@Controller`进行注解，这样组件扫描才能找到它。
- Spring Boot的`@EnableAutoConfiguration`：这个不起眼的小注解也可以称为`@Abracadabra`***{![abracadabra的意思是咒语。——译者注]}***，就是这一行配置开启了Spring Boot自动配置的魔力，让你不用再写成篇的配置了。

在Spring Boot的早期版本中，你需要在`ReadingListApplication`类上同时标上这三个注解，但从Spring Boot 1.2.0开始，有`@SpringBootApplication`就行了。

如我所说，`ReadingListApplication`还是一个启动引导类。要运行Spring Boot应用程序有几种方式，其中包含传统的WAR文件部署。但这里的`main()`方法让你可以在命令行里把该应用程序当作一个可执行JAR文件来运行。这里向`SpringApplication.run()`传递了一个`ReadingListApplication`类的引用，还有命令行参数，通过这些东西启动应用程序。

实际上，就算一行代码也没写，此时你仍然可以构建应用程序尝尝鲜。要构建并运行应用程序，最简单的方法就是用Gradle的`bootRun`任务：
```
$ gradle bootRun
```
`bootRun`任务来自Spring Boot的Gradle插件，我们会在2.1.2节里详细讨论。此外，你也可以用Gradle构建项目，然后在命令行里用`java`来运行它：
```
$ gradle build
...
$ java -jar build/libs/readinglist-0.0.1-SNAPSHOT.jar
```
应用程序应该能正常运行，启动一个监听8080端口的Tomcat服务器。要是愿意，你可以用浏览器访问[http://localhost:8080](http://localhost:8080)，但由于还没写控制器类，你只会收到一个HTTP 404 （Not Found）错误，看到错误页面。在本章结束前，这个URL将会提供一个阅读列表应用程序。

你几乎不需要修改`ReadingListApplication.java`。如果你的应用程序需要Spring Boot自动配置以外的其他Spring配置，一般来说，最好把它写到一个单独的`@Configuration`标注的类里。（组件扫描会发现并使用这些类的。）极度简单的情况下，可以把自定义配置加入`ReadingListApplication.java`。

#### 2.测试Spring Boot应用程序

Initializr还提供了一个测试类的骨架，可以基于它为你的应用程序编写测试。但`ReadingListApplicationTests`（代码清单2-2）不止是个用于测试的占位符，它还是一个例子，告诉你如何为Spring Boot应用程序编写测试。

**代码清单2-2 `@SpringApplicationConfiguration`加载Spring应用程序上下文**

>原书P28代码

>**文字**

>Load context via Spring Boot ：通过Spring Boot加载上下文

>Test that the context loads ：测试加载的上下文

一个典型的Spring集成测试会用`@ContextConfiguration`注解标识如何加载Spring的应用程序上下文。但是，为了充分发挥Spring Boot的魔力，这里应该用`@SpringApplicationConfiguration`注解。正如你在代码清单2-2里看到的那样，`ReadingListApplicationTests`使用`@SpringApplicationConfiguration`注解从`ReadingListApplication`配置类里加载Spring应用程序上下文。

`ReadingListApplicationTests`里还有一个简单的测试方法，即`contextLoads()`。实际上它就是个空方法。但这个空方法足以证明应用程序上下文的加载没有问题。如果`ReadingListApplication`里定义的配置是好的，就能通过测试。如果有问题，测试就会失败。

当然，现在这只是一个新的应用程序，你还会添加自己的测试。但`contextLoads()`方法是个良好的开端，此刻可以验证应用程序提供的各种功能。第4章会更详细地讨论如何测试Spring Boot应用程序。

####3.配置应用程序属性

Initializr为你生成的application.properties文件是一个空文件。实际上，这个文件完全是可选的，你大可以删掉它，这不会对应用程序有任何影响，但留着也没什么问题。

稍后，我们肯定有机会向application.properties里添加几个条目。但现在，如果你想小试牛刀，可以加一行看看：
```
server.port=8000
```
加上这一行，嵌入式Tomcat的监听端口就变成了8000，而不是默认的8080。你可以重新运行应用程序，看看是不是这样。

这说明application.properties文件可以很方便地帮你细粒度地调整Spring Boot的自动配置。你还可以用它来指定应用程序代码所需的配置项。在第3章里我们会看到好几个例子，演示`application.properties`的这两种用法。

要注意的是，你完全不用告诉Spring Boot为你加载`application.properties`，只要它存在就会被加载，Spring和应用程序代码都能获取其中的属性。

我们差不多已经把初始化的项目介绍完了，还剩最后一样东西，让我们来看看Spring Boot应用程序是如何构建的。

### 2.1.2 Spring Boot项目构建过程解析

Spring Boot应用程序的大部分内容都与其他Spring应用程序没有什么区别，与其他Java应用程序也没什么两样，因此构建一个Spring Boot应用程序和构建其他Java应用程序的过程类似。你可以选择Gradle或Maven作为构建工具，描述构建说明文件的方法和描述非Spring Boot应用程序的方法相似。但是，Spring Boot在构建过程中耍了些小把戏，在此需要做个小小的说明。

Spring Boot为Gradle和Maven提供了构建插件，以便辅助构建Spring Boot项目。代码清单2-3是Initializr创建的build.gradle文件，其中应用了Spring Boot的Gradle插件。

__代码清单2-3 使用Spring Boot的Gradle插件__

>原书P30

>**文字**

>Depend on Spring Boot plugin ：依赖Spring Boot插件

>Apply Spring Boot plugin ：运用Spring Boot插件

>Starter dependencies ：起步依赖

另一方面，要是选择用Maven来构建应用程序，Initializr会替你生成一个pom.xml文件，其中使用了Spring Boot的Maven插件，如代码清单2-4所示。

__代码清单2-4 使用Spring Boot的Maven插件及父起步依赖__

>原书P31代码

>**文字**

>Inherit versions from starter parent  ：从spring-boot-starter-parent继承版本号

>Starter dependencies ：起步依赖

>Apply Spring Boot plugin：  运用Spring Boot插件

无论你选择Gradle还是Maven，Spring Boot的构建插件都对构建过程有所帮助。你已经看到过如何用Gradle的`bootRun`任务来运行应用程序了。Spring Boot的Maven插件与之类似，提供了一个`spring-boot:run`目标，如果你使用Maven，它能实现相同的功能。

构建插件的主要功能是把项目打包成一个可执行的超级JAR（uber-JAR），包括把应用程序的所有依赖打入JAR文件内，并为JAR添加一个描述文件，其中的内容能让你用`java -jar`来运行应用程序。

除了构建插件，代码清单2-4里的Maven构建说明中还将spring-boot-starter-parent作为上一级，这样一来就能利用Maven的依赖管理功能，继承很多常用库的依赖版本，在你声明依赖时就不用再去指定版本号了。请注意，这个pom.xml里的`<dependency>`都没有指定版本。

遗憾的是，Gradle并没有Maven这样的依赖管理功能，为此Spring Boot Gradle插件提供了第三个特性，它为很多常用的Spring及其相关依赖模拟了依赖管理功能。其结果就是，代码清单2-3的build.gradle里也没有为各项依赖指定版本。

说起依赖，无论哪个构建说明文件，都只有五个依赖，除了你手工添加的H2之外，其他的Artifact ID都有spring-boot-starter-前缀。这些都是Spring Boot起步依赖，它们都有助于Spring Boot应用程序的构建。让我们来看看它们究竟都有哪些好处。
