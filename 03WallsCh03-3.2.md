## 3.2 通过属性文件外置配置

在处理应用安全时，你当然会希望完全掌控所有配置。不过，为了微调一些细节，比如改改端口号和日志级别，便放弃自动配置，这是一件让人羞愧的事。为了设置数据库URL，是配置一个属性简单，还是完整地声明一个数据源的Bean简单？答案不言而喻，不是吗？

事实上，Spring Boot自动配置的Bean提供了300多个用于微调的属性。当你调整设置时，只要在环境变量、Java系统属性、JNDI（Java Naming and Directory Interface）、命令行参数或者属性文件里进行指定就好了。

要了解这些属性，让我们来看个非常简单的例子。你也许已经注意到了，在命令行里运行阅读列表应用程序时，Spring Boot有一个ascii-art Banner。如果你想禁用这个Banner，可以将`spring.main.show-banner`属性设置为`false`。有几种实现方式，其中之一就是在运行应用程序的命令行参数里指定：
```
$ java -jar readinglist-0.0.1-SNAPSHOT.jar --spring.main.show-banner=false
```
另一种方式是创建一个名为application.properties的文件，包含如下内容：
```
spring.main.show-banner=false
```

或者，如果你喜欢的话，也可以创建名为application.yml的YAML文件，内容如下：
```
spring:
  main:
  show-banner: false
```
还可以将属性设置为环境变量。举例来说，如果你用的是bash或者zsh，可以用`export`命令：
```
$ export spring_main_show_banner=false
```
请注意，这里用的是下划线而不是点和横杠，这是对环境变量名称的要求。

实际上，Spring Boot应用程序有多种设置途径。Spring Boot能从多种属性源获得属性，包括如下几点。

1. 命令行参数
2. `java:comp/env`里的JNDI属性
3. JVM系统属性
4. 操作系统环境变量
5. 随机生成的带`random.*`前缀的属性（在设置其他属性时，可以引用它们，比如`${random.long}`）
6. 应用程序以外的application.properties或者appliaction.yml文件
7. 打包在应用程序内的application.properties或者appliaction.yml文件
8. 通过`@PropertySource`标注的属性源
9. 默认属性

这个列表按照优先级排序，也就是说，任何在高优先级属性源里设置的属性都会覆盖低优先级的相同属性。例如，命令行参数会覆盖其他属性源里的属性。

application.properties和application.yml文件能放在以下四个位置。

1. 外置，在相对于应用程序运行目录的/config子目录里。
2. 外置，在应用程序运行的目录里。
3. 内置，在config包内。
4. 内置，在Classpath根目录。

同样，这个列表按照优先级排序。也就是说，/config子目录里的application.properties会覆盖应用程序Classpath里的application.properties中的相同属性。

此外，如果你在同一优先级位置同时有application.properties和application.yml，那么application.yml里的属性会覆盖application.properties里的属性。

禁用ascii-art Banner只是使用属性的一个小例子。让我们再看几个例子，看看如何通过常用途径微调自动配置的Bean。

### 3.2.1 自动配置微调

如上所说，有300多个属性可以用来微调Spring Boot应用程序里的Bean。附录C有一个详尽的列表。此处无法逐一描述它们的细节，因此我们就通过几个例子来了解一些Spring Boot暴露的实用属性。

#### 1.禁用模板缓存

如果阅读列表应用程序经过了几番修改，你一定已经注意到了，除非重启应用程序，否则对Thymeleaf模板的变更是不会生效的。这是因为Thymeleaf模板默认缓存。这有助于改善应用程序的性能，因为模板只需编译一次，但在开发过程中就不能实时看到变更的效果了。

将`spring.thymeleaf.cache`设置为`false`就能禁用Thymeleaf模板缓存。在命令行里运行应用程序时，将其设置为命令行参数即可：
```
$ java -jar readinglist-0.0.1-SNAPSHOT.jar --spring.thymeleaf.cache=false
```
或者，如果你希望每次运行时都禁用缓存，可以创建一个application.yml，包含以下内容：
```
spring:
  thymeleaf:
    cache: false
```
你一定要确保这个文件不会发布到生产环境，否则生产环境里的应用程序就无法享受模板缓存带来的性能提升了。

作为开发者，在修改模板时始终关闭缓存实在太方便了。为此，可以通过环境变量来禁用Thymeleaf缓存：
```
$ export spring_thymeleaf_cache=false
```
即使我们使用Thymeleaf作为应用程序的视图，Spring Boot支持的其他模板也能关闭模板缓存，设置这些属性就好了：

-  `spring.freemarker.cache`（Freemarker）
- `spring.groovy.template.cache`（Groovy模板）
- `spring.velocity.cache`（Velocity）

默认情况下，这些属性都为`true`，也就是开启缓存。将它们设置为`false`即可禁用缓存。

#### 2.配置嵌入式服务器

从命令行（或者Spring Tool Suite）运行Spring Boot应用程序时，应用程序会启动一个嵌入式的服务器（默认是Tomcat），监听8080端口。大部分情况下这挺好的，但同时运行多个应用程序可能会有问题。要是所有应用程序都试着让Tomcat服务器监听同一个端口，在启动第二个应用程序时就会有冲突。

无论出于什么原因，让服务器监听不同的端口，你所要做的就是设置`server.port`属性。要是只改一次，可以用命令行参数：
```
$ java -jar readinglist-0.0.1-SNAPSHOT.jar --server.port=8000
```
但如果希望端口变更时间更长一点，可以在其他支持的配置位置上设置`server.port`。例如，把它放在应用程序Classpath根目录的application.yml文件里：
```
server:
  port: 8000
```
除了服务器的端口，你还可能希望服务器提供HTTPS服务。为此，第一步就是用JDK的`keytool`工具来创建一个密钥存储（keystore）：
```
$ keytool -keystore mykeys.jks -genkey -alias tomcat -keyalg RSA
```
该工具会询问几个与名字和组织相关的问题，大部分都无关紧要。但在被问到密码时，一定要记住你的选择。在本例中，我选择letmein作为密码。

现在只需要设置几个属性就能开启嵌入式服务器的HTTPS服务了。可以把它们都配置在命令行里，但这样太不方便了。可以把它们放在application.properties或application.yml里。在application.yml中，它们可能是这样的：
```
server:
  port: 8443
  ssl:
    key-store: file:///path/to/mykeys.jks
    key-store-password: letmein
    key-password: letmein
```
此处的`server.port`设置为8443，开发环境的HTTPS服务器大多会选这个端口。`server.ssl.key-store`属性指向密钥存储文件的存放路径。这里用了一个file://开头的URL，从文件系统里加载该文件。你也可以把它打包在应用程序的JAR文件里，用`classpath: URL`来引用它。`server.ssl.key-store-password`和`server.ssl.key-password`设置为创建该文件时给定的密码。

有了这些属性，应用程序就能在8443端口上监听HTTPS请求了。（根据你所用的浏览器，可能会出现警告框提示该服务器无法验证其身份。在开发时，访问的是localhost，这没什么好担心的。）

#### 3.配置日志

大多数应用程序都提供了某种形式的日志。即使你的应用程序不会直接记录日志，你所用的库也会记录它们的活动。

默认情况下，Spring Boot会用Logback（[http://logback.qos.ch](http://logback.qos.ch)）来记录日志，并用INFO级别输出到控制台。在运行应用程序和其他例子时，你应该已经看到很多INFO级别的日志了。

>__用其他日志实现替换Logback__

>一般来说，你不需要切换日志实现；Logback能很好地满足你的需要。但是，如果决定使用Log4j或者Log4j2，那么你只需要修改依赖，引入对应该日志实现的起步依赖，同时排除掉Logback。

>以Maven为例，应排除掉根起步依赖传递引入的默认日志起步依赖，这样就能排除Logback了：
>```
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter</artifactId>
  <exclusions>
    <exclusion>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-logging</artifactId>
    </exclusion>
  </exclusions>
</dependency>
```
>在Gradle里，在`configurations`下排除该起步依赖是最简单的办法：
>```
configurations {
  all*.exclude group:'org.springframework.boot',
               module:'spring-boot-starter-logging'
}
```
>排除默认日志的起步依赖后，就可以引入你想用的日志实现的起步依赖了。在Maven里可以这样添加Log4j：
>```
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-log4j</artifactId>
</dependency>
```
>在Gradle里可以这样添加Log4j：
>```
compile("org.springframework.boot:spring-boot-starter-log4j")
```
>如果你想用Log4j2，可以把spring-boot-starter-log4j改成spring-boot-starter-log4j2。

要完全掌握日志配置，可以在Classpath的根目录（src/main/resources）里创建logback.xml文件。下面是一个简单logback.xml的例子：
```
<configuration>
  <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
    <encoder>
      <pattern>
        %d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n
      </pattern>
    </encoder>
  </appender>

  <logger name="root" level="INFO"/>

  <root level="INFO">
    <appender-ref ref="STDOUT" />
  </root>
</configuration>
```
除了日志格式之外，这个Logback配置和不加logback.xml文件的默认配置差不多。但是，通过编辑logback.xml，你可以完全掌控应用程序的日志文件。哪些配置应该放进logback.xml这个话题不在本书的讨论范围内，请参考Logback的文档以了解更多信息。

即使如此，你对日志配置最常做的改动就是修改日志级别和指定日志输出的文件。使用了Spring Boot的配置属性后，你可以在不创建logback.xml文件的情况下修改那些配置。

要设置日志级别，你可以创建以`logging.level`开头的属性，后面是你想为其设置日志级别的日志名称。举例来说，假设你要把根日志级别设置为`WARN`，但Spring Security的日志要用`DEBUG`级别。在application.yml里加入以下内容就行了：
```
logging:
  level:
    root: WARN
    org:
      springframework:
        security: DEBUG
```
另外，你也可以把Spring Security的包名写成一行：
```
logging:
  level:
    root: WARN
    org.springframework.security: DEBUG
```
现在，假设你想把日志写到位于/var/logs/目录里的BookWorm.log文件里。使用`logging.path`和`loggin.file`属性就行了：
```
logging:
  path: /var/logs/
  file: BookWorm.log
  level:
    root: WARN
    org:
      springframework:
        security: DEBUG
```
假设应用程序有/var/logs/的写权限，日志就能被写入/var/logs/BookWorm.log。默认情况下，日志文件的大小达到10MB时会切分一次。

与之类似，这些属性也能在application.properties里设置：
```
logging.path=/var/logs/
logging.file=BookWorm.log
logging.level.root=WARN
logging.level.root.org.springframework.security=DEBUG
```
如果你还是想要完全掌控日志配置，但是又不想用logback.xml作为Logback配置的名字，可以通过`logging.config`属性指定自定义的名字：
```
logging:
  config:
    classpath:logging-config.xml
```
虽然一般并不需要改变配置文件的名字，但是如果你想针对不同运行时Profile使用不同的日志配置（见3.2.3节），这个功能会很有用。

#### 4.配置数据源

此时，我们还在开发阅读列表应用程序，嵌入式的H2数据库能很好地满足我们的需要。可是一旦要投放到生产环境，我们可能要考虑更持久的数据库解决方案。

虽然你可以显式配置自己的`DataSource` Bean，但通常并不用这么做，只需简单地通过属性配置数据库的URL和身份信息就可以了。举例来说，如果你用的是MySQL数据库，你的application.yml文件看起来可能是这样的：
```
spring:
  datasource:
    url: jdbc:mysql://localhost/readinglist
    username: dbuser
    password: dbpass
```
通常你都无需指定JDBC驱动，Spring Boot会根据数据库URL识别出需要的驱动，但如果识别出问题了，你还可以设置`spring.datasource.driver-class-name`属性：
```
spring:
  datasource:
    url: jdbc:mysql://localhost/readinglist
    username: dbuser
    password: dbpass
    driver-class-name: com.mysql.jdbc.Driver
```
在自动配置`DataSource` Bean的时候，Spring Boot会使用这里的连接数据。`DataSource` Bean是一个连接池，如果Classpath里有Tomcat的连接池`DataSource`，那么就会使用这个连接池；否则，Spring Boot会在Classpath里查找以下连接池：

- HikariCP
- Commons DBCP
- Commons DBCP 2

这里列出的只是自动配置支持的连接池，你还可以自己配置`DataSource` Bean，使用你喜欢的各种连接池。

你也可以设置`spring.datasource.jndi-name`属性，从JNDI里查找`DataSource`：
```
spring:
  datasource:
    jndi-name: java:/comp/env/jdbc/readingListDS
```
一旦设置了`spring.datasource.jndi-name`属性，其他数据源连接属性都会被忽略，除非没有设置别的数据源连接属性。

有很多影响Spring Boot自动配置组件的方法，只需设置一两个属性即可。但这种配置外置的方法并不局限于Spring Boot配置的Bean。让我们看看如何使用这种属性配置机制来微调自己的应用程序组件。

### 3.2.2 应用程序Bean的配置外置

假设我们在某人的阅读列表里不止想要展示图书标题，还要提供该书的Amazon链接。我们不仅想提供该书的链接，还要标记该书，以便利用Amazon的Associate Program，这样如果有人用我们应用程序里的链接买了书，我们还能收到一笔推荐费。

这很简单，只需修改Thymeleaf模板，以链接的形式来呈现每本书的标题就可以了：

```
<a th:href="'http://www.amazon.com/gp/product/'
            + ${book.isbn}
            + '/tag=habuma-20'"
   th:text="${book.title}">Title</a>
```
这样就好了。现在如果有人点击该链接并购买了本书，我就能得到推荐费了，因为habuma-20是我的Amazon Associate ID。如果你也想收到推荐费，可以把Thymeleaf模板中`tag`的值改成你的Amazon Associate ID。

虽然在模板里修改这个值很简单，但毕竟这也算是硬编码。我们现在只在这一个模板里链接到Amazon，但后续可能会有更多页面要链接到Amazon，于是需要为应用程序添加功能。那样的话，修改Amazon Associate ID就要改动好几个地方的代码。这就是为什么类似这种细节的东西最好不要放在代码里，要把它们集中在一个地方维护。

我们可以不在模板里硬编码Amazon Associate ID，而是把它变成模型中的一个值：
```
<a th:href="'http://www.amazon.com/gp/product/'
            + ${book.isbn}
            + '/tag=' + ${amazonID}"
   th:text="${book.title}">Title</a>
```  
此外，`ReadingListController`需要在模型里包含`amazonID`这个键，其中的内容是Amazon Associate ID。同样的道理，我们不应该硬编码这个值，而是应该引用一个实例变量。这个变量的值应该来自属性配置。代码清单3-4就是新的`ReadingListController`，它会返回注入的Amazon Associate ID。

__代码清单3-4 修改后的`ReadingListController`，能接受Amazon ID__

> 原书P65~66代码

>**文字**

> Inject with properties  ：属性注入

>Setter method for associateId  ：`associateId`的setter方法

>Put associateId into model  ：将`associateId`放入模型

如你所见，`ReadingListController`现在有了一个`associateId`属性，还有对应的`setAssociateId()`方法，用它可以设置该属性。`readersBooks()`现在能通过`amazonID`这个键把`associateId`的值放入模型。

棒极了！现在就剩一个问题——从哪里能取到`associateId`的值。

请注意，`ReadingListController`上加了`@ConfigurationProperties`注解，这说明该Bean的属性应该是（通过setter方法）从配置属性值注入的。说得更具体一点，`prefix`属性说明`ReadingListController`应该注入带`amazon`前缀的属性。

综合起来，我们指定`ReadingListController`的属性应该从带`amazon`前缀的配置属性中进行注入。`ReadingListController`只有一个setter方法，就是设置`associateId`属性用的setter方法。因此，设置Amazon Associate ID唯一要做的就是添加`amazon.associateId`属性，把它加入支持的任一属性源位置里即可。

例如，我们可以在application.properties里设置该属性：
```
amazon.associateId=habuma-20
```
或者在application.yml里：
```
amazon:
  associateId: habuma-20
```
或者，我们可以将其设置为环境变量，把它作为命令行参数，或把它加到任何能够设置配置属性的地方。

> __开启配置属性__

> 从技术上来说，`@ConfigurationProperties`注解不会生效，除非先向Spring配置类添加`@EnableConfigurationProperties`注解。但通常无需这么做，因为Spring Boot自动配置后面的全部配置类都已经加上了`@EnableConfigurationProperties`注解。因此，除非你完全不使用自动配置（那怎么可能？），否则就无需显式地添加`@EnableConfigurationProperties`。

还有一点需要注意，Spring Boot的属性解析器非常智能，它会自动把驼峰规则的属性和使用连字符或下划线的同名属性关联起来。换句话说，`amazon.associateId`这个属性和`amazon.associate_id`以及`amazon.associate-id`都是等价的。用你习惯的命名规则就好。

####在一个类里收集属性

虽然在`ReadingListController`上加上`@ConfigurationProperties`注解跑起来没问题，但这并不是一个理想的方案。`ReadingListController`和Amazon没什么关系，但属性的前缀却是`amazon`，这看起来难道不奇怪吗？再说，后续的各种功能可能需要在`ReadingListController`里新增配置属性，而它们和Amazon无关。

与其在`ReadingListController`里加载配置属性，还不如创建一个单独的Bean，为它加上`@ConfigurationProperties`注解，让这个Bean收集所有配置属性。代码清单3-5里的`AmazonProperties`就是一个例子，它用于加载Amazon相关的配置属性。

__代码清单3-5 在一个Bean里加载配置属性__

>原书P67

>**文字**

>Inject with “amazon”-prefixed properties：注入带amazon前缀的属性

>associateId setter method：associateId的setter方法

有了加载`amazon.associateId`配置属性的`AmazonProperties`后，我们可以调整`ReadingListController`（如代码清单3-6所示），让它从注入的`AmazonProperties`中获取Amazon Associate ID。

__代码清单3-6 注入了`AmazonProperties`的`ReadingListController`__

>原书P68

>**文字**

>Inject AmazonProperties：注入`AmazonProperties`

>Add Associate ID to model ：向模型中添加Associate ID

`ReadingListController`不再直接加载配置属性，转而通过注入其中的`AmazonProperties` Bean来获取所需的信息。

如你所见，配置属性在调优方面十分有用，这里说的调优不仅涵盖了自动配置的组件，还包括注入自有应用程序Bean的细节。但如果我们想为不同的部署环境配置不同的属性又该怎么办？让我们看看如何使用Spring的Profile来设置特定环境的配置。

### 3.2.3 使用Profile进行配置

当应用程序需要部署到不同的运行环境时，一些配置细节通常会有所不同。比如，数据库连接的细节在开发环境下和测试环境下就会不一样，在生产环境下又不一样。Spring Framework从Spring 3.1开始支持基于Profile的配置。Profile是一种条件化配置，基于运行时激活的Profile，会使用或者忽略不同的Bean或配置类。

举例来说，假设我们在代码清单3-1里创建的安全配置是针对生产环境的，而自动配置的安全配置用在开发环境刚刚好。在这个例子中，我们就能为`SecurityConfig`加上`@Profile`注解：
```
@Profile("production")
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

...

}
```

这里用的`@Profile`注解要求运行时激活`production` Profile，这样才能应用该配置。如果`production` Profile没有激活，就会忽略该配置，而此时缺少其他用于覆盖的安全配置，于是应用自动配置的安全配置。

设置`spring.profiles.active`属性就能激活Profile，任意设置配置属性的方式都能用于设置这个值。例如，在命令行里运行应用程序时，可以这样激活`production` Profile：
```
$ java -jar readinglist-0.0.1-SNAPSHOT.jar --
     spring.profiles.active=production
```
也可以向application.yml里添加`spring.profiles.active`属性：
```
spring:
  profiles:
    active: production
```
还可以设置环境变量，将其放入application.properties，或者使用3.2节开头提到的各种方法。

但由于Spring Boot的自动配置替你做了太多的事情，要找到一个能给你放`@Profile`的地方还真不怎么方便。幸运的是，Spring Boot支持为application.properties和application.yml里的属性配置Profile。

为了演示区分Profile的属性，假设你希望针对生产环境和开发环境能有不同的日志配置。在生产环境中，你只关心WARN或更高级别的日志项，想把日志写到日志文件里。在开发环境中，你只想把日志输出到控制台，记录DEBUG或更高级别。

而你所要做的就是为每个环境分别创建配置。那要怎么做呢？这取决于你用的是属性文件配置还是YAML配置。

#### 1.使用特定于Profile的属性文件

如果你正在使用application.properties，可以创建额外的属性文件，遵循application-{profile}.properties这种命名格式，这样就能提供特定于Profile的属性了。

在日志这个例子里，开发环境的配置可以放在名为application-development.properties的文件里，配置包含日志级别和输出到控制台：
```
logging.level.root=DEBUG
```
对于生产环境，application-production.properties会将日志级别设置为WARN或更高级别，并将日志写入日志文件：
```
logging.path=/var/logs/
logging.file=BookWorm.log
logging.level.root=WARN
```
与此同时，那些并不特定于哪个Profile或者保持默认值（以防万一有哪个特定于Profile的配置不指定这个值）的属性，可以继续放在application.properties里：
```
amazon.associateId=habuma-20
logging.level.root=INFO
```

#### 2.使用多Profile YAML文件进行配置

如果你使用YAML来配置属性，可以遵循与配置文件相同的命名规范，即创建application-{profile}.yml这样的YAML文件，并将与Profile无关的属性继续放在application.yml里。

但既然用了YAML，你就可以把所有Profile的配置属性都放在一个application.yml文件里。举例来说，我们可以像下面这样声明日志配置：
```
logging:
  level:
    root: INFO

---

spring:
  profiles: development

logging:
  level:
    root: DEBUG

---

spring:
  profiles: production

logging:
  path: /tmp/
  file: BookWorm.log
  level:
    root: WARN
```
如你所见，这个application.yml文件分为三个部分，使用一组三个连字符（`---`）作为分隔符。第二段和第三段分别为`spring.profiles`指定了一个值，这个值表示该部分配置应该应用在哪个Profile里。第二段中定义的属性应用于开发环境，因为`spring.profiles`设置为`development`。与之类似，最后一段的`spring.profile`设置为`production`，在`production` Profile被激活时生效。

另一方面，第一段并未指定`spring.profiles`，因此这里的属性对全部Profile都生效，或者对那些未设置该属性的激活Profile生效。

除了自动配置和外置配置属性，Spring Boot还有其他简化常用开发任务的绝招：它自动配置了一个错误页面，在应用程序遇到错误时显示。下一节，我们会介绍Spring Boot的错误页，以及如何定制这个错误页来适应我们的应用程序。
