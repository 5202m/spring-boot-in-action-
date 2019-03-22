# 第6章 在Spring Boot中使用Grails

>本章内容

>* 使用GORM持久化数据
* 定义GSP视图
* Grails 3和Spring Boot入门

我小时候，有一个系列电视广告，当中有两个人，一个在吃巧克力条，另一个在吃罐子里的花生酱。经由一些富有喜剧效果的小事故，两个人撞到了一起。最后，花生酱和巧克力相结合。

一个人说：“你把巧克力弄到我的花生酱里了！”另一个回答：“是你把花生酱弄到我的巧克力上了！”

在一开始的尴尬后，两个人都认同花生酱和巧克力结合在一起是件好事。接着，旁白会建议观众试试Reese牌的的花生酱杯（Peanut Butter Cup）。

在Spring Boot刚发布时，经常有人问我在Spring Boot和Grails之间该如何选择。两者都构建于Spring Framework之上，都旨在简化应用程序的开发。实际上，它们就像花生酱和巧克力。两个都很好，具体如何选择取决于个人爱好。

就像之前巧克力和花生酱的争论一样，事实上并不必从中选出一个来。Spring Boot和Grails两个都很好，完全可以结合到一起。

在本章中，我们会看到Grails和Spring Boot之间的联系。我们会先看到Spring Boot中Grails对象关系映射（Grails Object Relational Mapping，GORM）和Groovy服务器页面（Groovy Server Page，GSP）这样的Grails特性，还会看到Grails 3是如何基于Spring Boot重写的。

## 6.1 使用GORM进行数据持久化

Grails里最让人着迷的恐怕就是GORM了。GORM将数据库相关工作简化到和声明要持久化的实体一样容易。例如，代码清单6-1演示了阅读列表里的`Book`该如何用Groovy写成GORM实体。

__代码清单6-1 GORM `Book`实体__

>原书P108代码

>文字

>This is a GORM entity：这是一个GORM实体

就和`Book`的Java版本一样，这个类里有很多描述图书的属性。但又与Java版本不一样，这里没有分号、`public`或`private`修饰符、setter和getter方法或其他Java中常见的代码噪声。是Grails的`@Entity`注解让这个类变成了GORM实例。这个简单的实体可干了不少事，包括将对象映射到数据库，为`Book`添加持久化方法，通过这些方法可以存取图书。

要在Spring Boot项目里使用GORM，必须在项目里添加GORM依赖。在Maven中，`<dependency>`看起来是这样的：

```
<dependency>
  <groupId>org.grails</groupId>
  <artifactId>gorm-hibernate4-spring-boot</artifactId>
  <version>1.1.0.RELEASE</version>
</dependency>
```

一样的依赖，在Gradle里是这样的：

```
compile("org.grails:gorm-hibernate4-spring-boot:1.1.0.RELEASE")
```

这个库自带了一些Spring Boot自动配置，会自动配置所有支持GORM所需的Bean。你只管写代码就好了。

>
__GORM在Spring Boot里的另一个选择__

> 正如其名，`gorm-hibernate4-spring-boot`是通过Hibernate开启GORM数据持久化的。对很多项目而言，这很好。但如果你想用MongoDB，那你会对Spring Boot里的MongoDB GORM支持很感兴趣。

> 它的Maven依赖是这样的：

>```
<dependency>
  <groupId>org.grails</groupId>
  <artifactId>gorm-mongodb-spring-boot</artifactId>
  <version>1.1.0.RELEASE</version>
</dependency>
```

> 下面是相同的Gradle依赖：
```
compile("org.grails:gorm-mongodb-spring-boot:1.1.0.RELEASE")
```
GORM的工作原理要求实体类必须用Groovy来编写。我们已经在代码清单6-1里写了一个`Book`实体，下面再写一个`Reader`实体：

__代码清单6-2 GORM `Reader`实体__

>原书P109~110

>文字

>This is an entity ：这是一个实体

>Implement UserDetails：实现了`UserDetails`

现在，我们的阅读列表应用程序里有了两个GORM实体，我们需要重写剩下的应用程序来使用这两个实体。因为使用Groovy是如此令人愉悦（和Grails很像），所以其他类我们也会用Groovy来编写。

首先是`ReadingListController`，如代码清单6-3所示。

__代码清单6-3 Groovy的`ReadingListController`__

>原书P110~111

>文字

>Find all reader books：查找读者的全部图书

>Save a book：保存一本书

这个版本的`ReadingListController`和第3章里的相比，最明显的不同之处在于，它是用Groovy写的，没有Java的那些代码噪声。最重要的不同之处在于，无需再注入`ReadingListRepository`，它直接通过`Book`类型持久化。

在`readersBooks()`方法里，它调用了`Book`的`findAllByReader()`静态方法，传入了指定的读者信息。虽然代码清单6-1没有提供`findAllByReader()`方法，但这段代码仍然可以执行，因为GORM会为我们实现这个方法。

与之类似，`addToReadingList()`方法使用了静态方法`withTransaction()`和实例方法`save()`。这两个方法也是GORM提供的，用于将`Book`保存到数据库里。

我们所要做的就是声明一些属性，在`Book`上添加`@Entity`注解。如果你问我怎么看——我觉得这笔买卖很划算。

`SecurityConfig`也要做类似的修改，通过GORM而非`ReadingListRepository`来获取`Reader`。代码清单6-4就是新的`SecurityConfig`。

__代码清单6-4 Groovy版本的`SecurityConfig`__

>原书P111~112代码

>文字

>Find a reader by username：根据用户名查找读者

除了用Groovy重写，`SecurityConfig`里最明显的变化无疑就是第二个`configure()`方法。如你所见，它使用了一个闭包（`UserDetailsService`的实现类），其中调用静态方法`findByUsername()`来查找`Reader`，这个功能是GORM提供的。

你也许会好奇——在这个GORM版本的应用程序里，`ReadingListRepository`变成什么了？GORM替我们处理了所有的持久化工作，这里已经不再需要`ReadingListRepository`了，它的实现也都不需要了。我想你会同意代码越少越好这个观点。

应用程序中剩余的代码也应该用Groovy重写，这样才能和我们的变更相匹配。但它们和GORM没什么关系，也不在本章的讨论范围内。如果想要完整的代码，可以到示范代码页面里去下载。

此刻，你可以通过各种运行Spring Boot应用程序的方法来启动阅读列表应用程序。启动后，应用程序应该能像从前一样工作。只有你我知道持久化机制已经被改变了。

除了GORM，Grails应用程序通常还会用Groovy Server Pages将模型数据以HTML的方式呈现给浏览器。6.2节应用程序的Grails化还会继续。我们会把Thymeleaf替换为等价的GSP。

| [PREV](https://github.com/5202m/spring-boot-in-action-zh-cn/blob/master/05WallsCh05-5.4.md) | [NEXT](https://github.com/5202m/spring-boot-in-action-zh-cn/blob/master/06WallsCh06-6.2.md) |
