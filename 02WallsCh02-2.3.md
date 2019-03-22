## 2.3 使用自动配置

简而言之，Spring Boot的自动配置是一个运行时（更准确地说，是应用程序启动时）的过程，考虑了众多因素，才决定Spring配置应该用哪个，不该用哪个。举几个例子，下面这些情况都是Spring Boot的自动配置要考虑的。

- Spring的`JdbcTemplate`是不是在Classpath里？如果是，并且有`DataSource`的Bean，就自动配置一个`JdbcTemplate`的Bean。
- Thymeleaf是不是在Classpath里？如果是的话，配置Thymeleaf的模板解析器、视图解析器以及模板引擎。
- Spring Security是不是在Classpath里？如果是的话，做一个非常基本的Web安全设置。

每当应用程序启动的时候，Spring Boot的自动配置都要做将近200个这样的决定，涵盖安全、集成、持久化、Web开发等诸多方面。所有这些自动配置就是为了尽量不让你自己写配置。

有意思的是，自动配置的东西很难写在书本里。如果不能写出配置，那又该怎么描述并讨论它们呢？

### 2.3.1 专注于应用程序功能

要为Spring Boot的自动配置博得好感，我可以在接下来的几页里向你演示没有Spring Boot的情况下需要写哪些配置。但眼下已经有不少好书写过这些内容了，再写一次并不能让我们更快地写好阅读列表应用程序。

既然知道Spring Boot会替我们料理这些事情，那么与其浪费时间讨论这些Spring配置，还不如看看如何利用Spring Boot的自动配置，让我们专注于应用程序代码。除了开始写代码，我想不到更好的办法了。

#### 1.定义领域模型

我们应用程序里的核心领域概念是读者阅读列表上的书。因此我们需要定义一个实体类来表示这个概念。代码清单2-5演示了如何定义一本书。

__代码2-5 表示列表里的书的`Book`类__
```
package readinglist;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;

@Entity
public class Book {

  @Id
  @GeneratedValue(strategy=GenerationType.AUTO)
  private Long id;
  private String reader;
  private String isbn;
  private String title;
  private String author;
  private String description;

  public Long getId() {
    return id;
  }

  public void setId(Long id) {
    this.id = id;
  }

  public String getReader() {
    return reader;
  }

  public void setReader(String reader) {
    this.reader = reader;
  }

  public String getIsbn() {
    return isbn;
  }

  public void setIsbn(String isbn) {
    this.isbn = isbn;
  }

  public String getTitle() {
    return title;
  }

  public void setTitle(String title) {
    this.title = title;
  }

  public String getAuthor() {
    return author;
  }

  public void setAuthor(String author) {
    this.author = author;
  }

  public String getDescription() {
    return description;
  }

  public void setDescription(String description) {
    this.description = description;
  }
}
```
如你所见，`Book`类就是简单的Java对象，其中有些描述书的属性，还有必要的访问方法。`@Entity`注解表明它是一个JPA实体，`id`属性加了`@Id`和`@GeneratedValue`注解，说明这个字段是实体的唯一标识，并且这个字段的值是自动生成的。

#### 2.定义仓库接口

接下来，我们就要定义用于把`Book`对象持久化到数据库的仓库了。***{![原文这里写的是`ReadingList`对象，但文中并没有定义这个对象，看代码应该是`Book`对象。——译者注]}***因为用了Spring Data JPA，所以我们要做的就是简单地定义一个接口，扩展一下Spring Data JPA的`JpaRepository`接口：
```
package readinglist;

import java.util.List;
import org.springframework.data.jpa.repository.JpaRepository;

public interface ReadingListRepository extends JpaRepository<Book, Long> {
  List<Book> findByReader(String reader);
}
```

通过扩展`JpaRepository`，`ReadingListRepository`直接继承了18个执行常用持久化操作的方法。`JpaRepository`是个泛型接口，有两个参数：仓库操作的领域对象类型，及其ID属性的类型。此外，我还增加了一个`findByReader()`方法，可以根据读者的用户名来查找阅读列表。

如果你好奇谁来实现这个`ReadingListRepository`及其继承的18个方法，请不用担心，Spring Data提供了很神奇的魔法，只需定义仓库接口，在应用程序启动后，该接口在运行时会自动实现。

#### 3.创建Web界面

现在，我们定义好了应用程序的领域模型，还有把领域对象持久化到数据库里的仓库接口，剩下的就是创建Web前端了。代码清单2-6的Spring MVC控制器就能为应用程序处理HTTP请求。

__代码清单2-6 作为阅读列表应用程序前端的Spring MVC控制器__

```
package readinglist;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

import java.util.List;

@Controller
@RequestMapping("/")
public class ReadingListController {
  private ReadingListRepository readingListRepository;

  @Autowired
  public ReadingListController(
             ReadingListRepository readingListRepository) {
    this.readingListRepository = readingListRepository;
  }

  @RequestMapping(value="/{reader}", method=RequestMethod.GET)
  public String readersBooks(
      @PathVariable("reader") String reader,
      Model model) {
    List<Book> readingList =
        readingListRepository.findByReader(reader);
    if (readingList != null) {
      model.addAttribute("books", readingList);
    }
    return "readingList";
  }

  @RequestMapping(value="/{reader}", method=RequestMethod.POST)
  public String addToReadingList(
            @PathVariable("reader") String reader, Book book) {
    book.setReader(reader);
    readingListRepository.save(book);
    return "redirect:/{reader}";
  }
}
```
`ReadingListController`使用了`@Controller`注解，这样组件扫描会自动将其注册为Spring应用程序上下文里的一个Bean。它还用了`@RequestMapping`注解，将其中所有的处理器方法都映射到了“/”这个URL路径上。

该控制器有两个方法。

- `readersBooks()`：处理/{reader}上的HTTP `GET`请求，根据路径里指定的读者，从（通过控制器的构造器注入的）仓库获取`Book`列表。随后将这个列表塞入模型，用的键是`books`，最后返回`readingList`作为呈现模型的视图逻辑名称。
- `addToReadingList()`：处理/{reader}上的HTTP `POST`请求，将请求正文里的数据绑定到一个`Book`对象上。该方法把`Book`对象的`reader`属性设置为读者的姓名，随后通过仓库的`save()`方法保存修改后的`Book`对象，最后重定向到/{reader}（控制器中的另一个方法会处理该请求）。

`readersBooks()`方法最后返回`readingList`作为逻辑视图名，为此必须创建该视图。因为在项目开始之初我就决定要用Thymeleaf来定义应用程序的视图，所以接下来就在src/main/resources/templates里创建一个名为readingList.html的文件，内容如代码清单2-7所示。

__代码清单2-7 呈现阅读列表的Thymeleaf模板__
```
<html>
  <head>
    <title>Reading List</title>
    <link rel="stylesheet" th:href="@{/style.css}"></link>
  </head>
  <body>
    <h2>Your Reading List</h2>
    <div th:unless="${#lists.isEmpty(books)}">
      <dl th:each="book : ${books}">
        <dt class="bookHeadline">
          <span th:text="${book.title}">Title</span> by
          <span th:text="${book.author}">Author</span>
          (ISBN: <span th:text="${book.isbn}">ISBN</span>)
        </dt>
        <dd class="bookDescription">
          <span th:if="${book.description}"
                th:text="${book.description}">Description</span>
          <span th:if="${book.description eq null}">
                No description available</span>
        </dd>
      </dl>
    </div>
    <div th:if="${#lists.isEmpty(books)}">
      <p>You have no books in your book list</p>
    </div>

    <hr/>

    <h3>Add a book</h3>
    <form method="POST">
      <label for="title">Title:</label>
        <input type="text" name="title" size="50"></input><br/>
      <label for="author">Author:</label>
        <input type="text" name="author" size="50"></input><br/>
      <label for="isbn">ISBN:</label>
        <input type="text" name="isbn" size="15"></input><br/>
      <label for="description">Description:</label><br/>
        <textarea name="description" cols="80" rows="5">
        </textarea><br/>
      <input type="submit"></input>
    </form>

  </body>
</html>
```

这个模板定义了一个HTML页面，该页面概念上分为两个部分：页面上方是读者的阅读列表中的图书清单；下方是是一个表单，读者可以从这里添加新书。

为了美观，Thymeleaf模板引用了一个名为style.css的样式文件，该文件位于src/main/resources/static目录中，看起来是这样的：

```
body {
    background-color: #cccccc;
    font-family: arial,helvetica,sans-serif;
}
.bookHeadline {
    font-size: 12pt;
    font-weight: bold;
}
.bookDescription {
    font-size: 10pt;
}
label {
    font-weight: bold;
}
```

这个样式表并不复杂，也没有过分追求让应用程序变漂亮，但已经能满足我们的需求了。很快你就会看到，它能用来演示Spring Boot的自动配置功能。

不管你相不相信，以上就是一个完整的应用程序了——本章已经向你呈现了所有的代码。等一下，回顾一下前几页的内容，你看到什么配置了吗？实际上，除了代码清单2-1里的三行配置（这是开启自动配置所必需的），你不用再写任何Spring配置了。

虽然没什么Spring配置，但是这已经是一个可以运行的完整Spring应用程序了。让我们把它跑起来，看看是怎么样的。

### 2.3.2 运行应用程序

运行Spring Boot应用程序有几种方法。先前在2.5节里，我们讨论了如何通过Maven和Gradle来运行应用程序，以及如何构建并运行可执行JAR。稍后，在第8章里你将看到如何构建WAR文件，并用传统的方式部署到Java Web应用服务器里，比如Tomcat。

假设你正使用Spring Tool Suite开发应用程序，可以在IDE里选中项目，在Run菜单里选择Run As > Spring Boot App，通过这种方式来运行应用程序，如图2-3所示。

> P43 ![Image text](https://raw.githubusercontent.com/5202m/spring-boot-in-action-zh-cn/master/imgs/figure-2.3.png)

__图2-3 在Spring Tool Suite里运行Spring Boot应用程序__

如果一切正常，你的浏览器应该会展现一个空白的阅读列表，下方有一个用于向列表添加新书的表单，如图2-4所示。

>P44 ![Image text](https://raw.githubusercontent.com/5202m/spring-boot-in-action-zh-cn/master/imgs/figure-2.4.png)

__图2-4 初始状态下的空阅读列表__

接下来，通过表单添加一些图书吧。随后你的阅读列表看起来就会像图2-5这样。

>P44 ![Image text](https://raw.githubusercontent.com/5202m/spring-boot-in-action-zh-cn/master/imgs/figure-2.5.png)

__图2-5 添加了一些图书后的阅读列表__

再多用用这个应用程序吧。你准备好之后，我们就来看一下Spring Boot是如何做到不写Spring配置代码就能开发整个Spring应用程序的。

### 2.3.3 刚刚发生了什么

如我所说，在没有配置代码的情况下，很难描述自动配置。与其花时间讨论那些你不用做的事情，不如在这一节里关注一下你要做的事——写代码。

当然，某处肯定是有些配置的。配置是Spring Framework的核心元素，必须要有东西告诉Spring如何运行应用程序。

在向应用程序加入Spring Boot时，有个名为spring-boot-autoconfigure的JAR文件，其中包含了很多配置类。每个配置类都在应用程序的Classpath里，都有机会为应用程序的配置添砖加瓦。这些配置类里有用于Thymeleaf的配置，有用于Spring Data JPA的配置，有用于Spiring MVC的配置，还有很多其他东西的配置，你可以自己选择是否在Spring应用程序里使用它们。

所有这些配置如此与众不同，原因在于它们利用了Spring的条件化配置，这是Spring 4.0引入的新特性。条件化配置允许配置存在于应用程序中，但在满足某些特定条件之前都忽略这个配置。

在Spring里可以很方便地编写你自己的条件，你所要做的就是实现`Condition`接口，覆盖它的`matches()`方法。举例来说，下面这个简单的条件类只有在Classpath里存在`JdbcTemplate`时才会生效：
```java
package readinglist;
import org.springframework.context.annotation.Condition;
import org.springframework.context.annotation.ConditionContext;
import org.springframework.core.type.AnnotatedTypeMetadata;

public class JdbcTemplateCondition implements Condition {
  @Override
  public boolean matches(ConditionContext context,
                         AnnotatedTypeMetadata metadata) {
    try {
      context.getClassLoader().loadClass(
             "org.springframework.jdbc.core.JdbcTemplate");
      return true;
    } catch (Exception e) {
      return false;
    }
  }
}
```
当你用Java来声明Bean的时候，可以使用这个自定义条件类：
```java
@Conditional(JdbcTemplateCondition.class)
public MyService myService() {
    ...
}
```
在这个例子里，只有当`JdbcTemplateCondition`类的条件成立时才会创建`MyService`这个Bean。也就是说`MyService` Bean创建的条件是Classpath里有`JdbcTemplate`。否则，这个Bean的声明就会被忽略掉。

虽然本例中的条件相当简单，但Spring Boot定义了很多更有趣的条件，并把它们运用到了配置类上，这些配置类构成了Spring Boot的自动配置。Spring Boot运用条件化配置的方法是，定义多个特殊的条件化注解，并将它们用到配置类上。表2-1列出了Spring Boot提供的条件化注解。

__表2-1 自动配置中使用的条件化注解__

| 条件化注解                       | 配置生效条件                                                            |
|---------------------------------|-------------------------------------------------------------------------|
| `@ConditionalOnBean`              | 配置了某个特定Bean                                                       |
| `@ConditionalOnMissingBean`       | 没有配置特定的Bean                                                       |
| `@ConditionalOnClass`             | Classpath里有指定的类                                                    |
| `@ConditionalOnMissingClass`      | Classpath里缺少指定的类                                                  |
| `@ConditionalOnExpression`        | 给定的Spring Expression Language（SpEL）表达式计算结果为`true`             |
| `@ConditionalOnJava`              | Java的版本匹配特定值或者一个范围值                                         |
| `@ConditionalOnJndi`              | 参数中给定的JNDI位置必须存在一个，如果没有给参数，则要有JNDI `InitialContext` |
| `@ConditionalOnProperty`          | 指定的配置属性要有一个明确的值                                             |
| `@ConditionalOnResource`          | Classpath里有指定的资源                                                   |
| `@ConditionalOnWebApplication`    | 这是一个Web应用程序                                                       |
| `@ConditionalOnNotWebApplication` | 这不是一个Web应用程序                                                     |

一般来说，无需查看Spring Boot自动配置类的源代码，但为了演示如何使用表2-1里的注解，我们可以看一下`DataSourceAutoConfiguration`里的这个片段（这是Spring Boot自动配置库的一部分）：
```
@Configuration
@ConditionalOnClass({ DataSource.class, EmbeddedDatabaseType.class })
@EnableConfigurationProperties(DataSourceProperties.class)
@Import({ Registrar.class, DataSourcePoolMetadataProvidersConfiguration.class })
public class DataSourceAutoConfiguration {
...
}
```

如你所见，`DataSourceAutoConfiguration`添加了`@Configuration`注解，它从其他配置类里导入了一些额外配置，还自己定义了一些Bean。最重要的是，`DataSourceAutoConfiguration`上添加了`@ConditionalOnClass`注解，要求Classpath里必须要有`DataSource`和`EmbeddedDatabaseType`。如果它们不存在，条件就不成立，`DataSourceAutoConfiguration`提供的配置都会被忽略掉。

`DataSourceAutoConfiguration`里嵌入了一个`JdbcTemplateConfiguration`类，自动配置了一个`JdbcTemplate` Bean：
```
@Configuration
@Conditional(DataSourceAutoConfiguration.DataSourceAvailableCondition.class)
protected static class JdbcTemplateConfiguration {

  @Autowired(required = false)
  private DataSource dataSource;

  @Bean
  @ConditionalOnMissingBean(JdbcOperations.class)
  public JdbcTemplate jdbcTemplate() {
    return new JdbcTemplate(this.dataSource);
  }

...

}
```
`JdbcTemplateConfiguration`使用了`@Conditional`注解，判断`DataSourceAvailableCondition`条件是否成立——基本上就是要有一个`DataSource` Bean或者要自动配置创建一个。假设有`DataSource` Bean，使用了`@Bean`注解的`jdbcTemplate()`方法会配置一个`JdbcTemplate` Bean。这个方法上还加了`@ConditionalOnMissingBean`注解，因此只有在不存在`JdbcOperations`（即`JdbcTemplate`实现的接口）类型的Bean时，才会创建`JdbcTemplate` Bean。

此处看到的只是`DataSourceAutoConfiguration`的冰山一角，Spring Boot提供的其他自动配置类也有很多知识没有提到。但这已经足以说明Spring Boot如何利用条件化配置实现自动配置。

自动配置会做出以下配置决策，它们和之前的例子息息相关。

- 因为Classpath里有H2，所以会创建一个嵌入式的H2数据库Bean，它的类型是`javax.sql.DataSource`，JPA实现（Hibernate）需要它来访问数据库。
- 因为Classpath里有Hibernate（Spring Data JPA传递引入的）的实体管理器，所以自动配置会配置与Hibernate相关的Bean，包括Spring的`LocalContainerEntityManagerFactoryBean`和`JpaVendorAdapter`。
- 因为Classpath里有Spring Data JPA，所以它会自动配置为根据仓库的接口创建仓库实现。
- 因为Classpath里有Thymeleaf，所以Thymeleaf会配置为Spring MVC的视图，包括一个Thymeleaf的模板解析器、模板引擎及视图解析器。视图解析器会解析相对于Classpath根目录的/templates目录里的模板。
- 因为Classpath里有Spring MVC（归功于Web起步依赖），所以会配置Spring的`DispatcherServlet`并启用Spring MVC。
- 因为这是一个Spring MVC Web应用程序，所以会注册一个资源处理器，把相对于Classpath根目录的/static目录里的静态内容提供出来。（这个资源处理器还能处理/public、/resources和/META-INF/resources的静态内容。）
- 因为Classpath里有Tomcat（通过Web起步依赖传递引用），所以会启动一个嵌入式的Tomcat容器，监听8080端口。

由此可见，Spring Boot自动配置承担起了配置Spring的重任，因此你能专注于编写自己的应用程序。
