## 2.2 使用起步依赖

要理解Spring Boot起步依赖带来的好处，先让我们假设它们尚不存在。如果没用Spring Boot的话，你会向项目里添加哪些依赖呢？要用Spring MVC的话，你需要哪个Spring依赖？你还记得Thymeleaf的Group和Artifact ID吗？你应该用哪个版本的Spring Data JPA呢？它们放在一起兼容吗？

看来如果没有Spring Boot起步依赖，你就有不少功课要做。而你想要做的只不过是开发一个Spring Web应用程序，使用Thymeleaf视图，通过JPA进行数据持久化。但在开始编写第一行代码之前，你得搞明白，要支持你的计划，需要往构建说明里加入哪些东西。

考虑再三之后（也许你还从其他有相似依赖的应用程序构建说明中复制粘贴了不少内容），你的Gradle构建说明里大概会有下面这些东西：
```
compile("org.springframework:spring-web:4.1.6.RELEASE")
compile("org.thymeleaf:thymeleaf-spring4:2.1.4.RELEASE")
compile("org.springframework.data:spring-data-jpa:1.8.0.RELEASE")
compile("org.hibernate:hibernate-entitymanager:jar:4.3.8.Final")
compile("com.h2database:h2:1.4.187")
```
这段依赖列表不错，应该能正常工作，但你是怎么知道的？你怎么保证你选的这些版本能相互兼容？也许可以，但构建并运行应用程序之前你是不知道的。再说了，你怎么知道这个列表是完整的？在一行代码都没写的情况下，你离开始构建还有很长的路要走。

让我们退一步再想想，我们要做什么。我们要构建一个拥有如下功能的应用程序。

- 这是一个Web应用程序。
- 它用了Thymeleaf。
- 它通过Spring Data JPA在关系型数据库里持久化数据。

如果我们只在构建文件里指定这些功能，让构建过程自己搞明白我们要什么东西，岂不是更简单？这正是Spring Boot起步依赖的功能。

### 2.2.1 指定基于功能的依赖

Spring Boot通过提供众多起步依赖降低项目依赖的复杂度。起步依赖本质上是一个Maven项目对象模型（Project Object Model，POM），定义了对其他库的传递依赖，这些东西加在一起即支持某项功能。很多起步依赖的命名都暗示了它们提供的某种或某类功能。

举例来说，你打算把这个阅读列表应用程序做成一个Web应用程序。与其向项目的构建文件里添加一堆单独的库依赖，还不如声明这是一个Web应用程序来得简单。你只要添加Spring Boot的Web起步依赖就好了。

我们还想以Thymeleaf为Web视图，用JPA来实现数据持久化，因此在构建文件里还需要Thymeleaf和Spring Data JPA的起步依赖。

为了能进行测试，我们还需要能在Spring Boot上下文里运行集成测试的库，因此要添加Spring Boot的test起步依赖，这是一个测试时依赖。

统统放在一起，我们就有了这五个依赖，也就是Initializr在Gradle的构建文件里提供的：
```
dependencies {
  compile "org.springframework.boot:spring-boot-starter-web"
  compile "org.springframework.boot:spring-boot-starter-thymeleaf"
  compile "org.springframework.boot:spring-boot-starter-data-jpa"
  compile "com.h2database:h2"
  testCompile("org.springframework.boot:spring-boot-starter-test")
}
```
正如先前所见，添加这些依赖的最简单方法就是在Initializr里选中Web、Thymeleaf和JPA复选框。但如果在初始化项目时没有这么做，当然也可以稍后再编辑生成的build.gradle或pom.xml。

通过传递依赖，添加这四个依赖就等价于加了一大把独立的库。这些传递依赖涵盖了Spring MVC、Spring Data JPA、Thymeleaf等内容，它们声明的依赖也会被传递依赖进来。

最值得注意的是，这四个起步依赖的具体程度恰到好处。我们并没有说想要Spring MVC，只是说想要构建一个Web应用程序。我们并没有指定JUnit或其他测试工具，只是说我们想要测试自己的代码。Thymeleaf和Spring Data JPA的起步依赖稍微具体一点，但这也只是由于没有更模糊的方法声明这种需要。

这四个起步依赖只是Spring Boot众多起步依赖中的沧海一粟。附录B罗列出了全部起步依赖，并简要描述了一下它们向项目构建引入了什么。

我们并不需要指定版本号，起步依赖本身的版本是由正在使用的Spring Boot的版本来决定的，而起步依赖则会决定它们引入的传递依赖的版本。

不知道自己所用依赖的版本，你多少会有些不安。你要有信心，相信Spring Boot经过了足够的测试，确保引入的全部依赖都能相互兼容。这是一种解脱，只需指定起步依赖，不用担心自己需要维护哪些库，也不必担心它们的版本。

但如果你真想知道自己在用什么，在构建工具里总能找到你要的答案。在Gradle里，`dependencies`任务会显示一个依赖树，其中包含了项目所用的每一个库以及它们的版本：
```
$ gradle dependencies
```
在Maven里使用`dependency`插件的`tree`目标也能获得相似的依赖树。
```
$ mvn dependency:tree
```
大部分情况下，你都无需关心每个Spring Boot起步依赖分别声明了些什么东西。Web起步依赖能让你构建Web应用程序，Thymeleaf起步依赖能让你用Thymeleaf模板，Spring Data JPA起步依赖能让你用Spring Data JPA将数据持久化到数据库里，通常只要知道这些就足够了。

但是，即使经过了Spring Boot团队的测试，起步依赖里所选的库仍有问题该怎么办？如何覆盖起步依赖呢？

### 2.2.2 覆盖起步依赖引入的传递依赖

说到底，起步依赖和你项目里的其他依赖没什么区别。也就是说，你可以通过构建工具中的功能，选择性地覆盖它们引入的传递依赖的版本号，排除传递依赖，当然还可以为那些Spring Boot起步依赖没有涵盖的库指定依赖。

以Spring Boot的Web起步依赖为例，它传递依赖了Jackson JSON库。如果你正在构建一个生产或消费JSON资源表述的REST服务，那它会很有用。但是，要构建传统的面向人类用户的Web应用程序，你可能用不上Jackson。虽然把它加进来也不会有什么坏处，但排除掉它的传递依赖，可以为你的项目瘦身。

如果在用Gradle，你可以这样排除传递依赖：
```
compile("org.springframework.boot:spring-boot-starter-web") {
  exclude group: 'com.fasterxml.jackson.core'
}
```
在Maven里，可以用`<exclusions>`元素来排除传递依赖。下面这个引入Spring Boot的build.gradle的`<dependency>`增加了`<exclusions>`元素去除Jackson：
```
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
  <exclusions>
    <exclusion>
      <groupId>com.fasterxml.jackson.core</groupId>
    </exclusion>
  </exclusions>
</dependency>
```
另一方面，也许项目需要Jackson，但你需要用另一个版本的Jackson来进行构建，而不是Web起步依赖里的那个。假设Web起步依赖引用了Jackson 2.3.4，但你需要使用2.4.3***{![此处提到的版本仅作演示之用，Spring Boot的Web起步依赖所引用的实际Jackson版本由你使用的Spring Boot版本决定。]}***。在Maven里，你可以直接在pom.xml中表达诉求，就像这样：
```
<dependency>
  <groupId>com.fasterxml.jackson.core</groupId>
  <artifactId>jackson-databind</artifactId>
  <version>2.4.3</version>
</dependency>
```
Maven总是会用最近的依赖，也就是说，你在项目的构建说明文件里增加的这个依赖，会覆盖传递依赖引入的另一个依赖。

与之类似，如果你用的是Gradle，可以在build.gradle文件里指明你要的Jackson的版本：
```
compile("com.fasterxml.jackson.core:jackson-databind:2.4.3")
```
因为这个依赖的版本比Spring Boot的Web起步依赖引入的要新，所以在Gradle里是生效的。但假如你要的不是新版本的Jackson，而是一个较早的版本呢？Gradle和Maven不太一样，Gradle倾向于使用库的最新版本。因此，如果你要使用老版本的Jackon，则不得不把老版本的依赖加入构建，并把Web起步依赖传递依赖的那个版本排除掉：

```
compile("org.springframework.boot:spring-boot-starter-web") {
  exclude group: 'com.fasterxml.jackson.core'
}
compile("com.fasterxml.jackson.core:jackson-databind:2.3.1")
```
不管什么情况，在覆盖Spring Boot起步依赖引入的传递依赖时都要多加小心。虽然不同的版本放在一起也许没什么问题，但你要知道，起步依赖中各个依赖版本之间的兼容性都经过了精心的测试。应该只在特殊的情况下覆盖这些传递依赖（比如新版本修复了一个bug）。

现在我们有了一个空的项目结构，构建说明文件也准备好了，是时候开发应用程序了。我们会让Spring Boot来处理配置细节，而我们自己则专注于编写阅读列表功能相关的代码。
