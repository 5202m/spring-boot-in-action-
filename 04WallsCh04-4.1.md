# 第4章 测试

>本章内容

>* 集成测试
* 在服务器里测试应用程序
* Spring Boot的测试辅助工具

有人说，如果你不知道要去哪，走就是了。但在软件开发领域，如果你没有目标，那结果往往是开发出一个满是bug的应用程序，没人用得了。

在编写应用程序时，明确目标的最佳方法就是写测试，确定应用程序的行为是否符合预期。如果测试失败了，你就有活要干了。如果测试通过了，那你就成功了（至少在你觉得还有其他测试要写之前，是这样的）。

究竟是在编写业务代码之前还是之后写测试，这并不重要。重要的是，写测试不仅仅是为了验证代码的准确性，还要确认它符合预期。测试也是一道保障，确认应用程序在改进的同时不会破坏已有的东西。

在编写单元测试的时候，Spring通常不需要介入。Spring鼓励松耦合、接口驱动的设计，这些都能让你很轻松地编写单元测试。但是在写单元测试时并不需要用到Spring。

但是，集成测试要用到Spring。如果生产应用程序使用Spring来配置并组装组件，那么测试就需要用它来配置并组装那些组件。

Spring的`SpringJUnit4ClassRunner`可以在基于JUnit的应用程序测试里加载Spring应用程序上下文。在测试Spring Boot应用程序时，Spring Boot除了拥有Spring的集成测试支持，还开启了自动配置和Web服务器，并提供了不少实用的测试辅助工具。

在本章中，我们会看到Spring Boot的各种集成测试支持。让我们先来看看如何在Spring Boot应用程序上下文里做测试。

## 4.1 集成测试自动配置

Spring Framework多项工作的核心是将所有组件编织在一起，构成一个应用程序。整个过程就是读取配置说明（可以是XML、基于Java的配置、基于Groovy的配置或其他类型的配置），在应用程序上下文里初始化Bean，将Bean注入依赖它们的其他Bean中。

对Spring应用程序进行集成测试时，让Spring遵照生产环境来组装测试目标Bean是非常重要的一点。当然，你也可以手动初始化组件，并将它们注入其他组件，但对那些大型应用程序来说，这是项费时费力的工作。而且，Spring提供了额外的辅助功能，比如组件扫描、自动织入和声明性切面（缓存、事务和安全，等等）。你要把这些活都干了，基本也就是把Spring再造了一次，最好还是让Spring替你把重活都做了吧，哪怕是在集成测试里。

Spring自1.1.1版就向集成测试提供了极佳的支持。自Spring 2.5开始，集成测试支持的形式就变成了`SpringJUnit4ClassRunner`。这是一个JUnit类运行器，会为JUnit测试加载Spring应用程序上下文，并为测试类自动织入所需的Bean。

举例来说，看一下代码清单4-1，这是一个非常基本的Spring集成测试。

__代码清单4-1 用`SpringJUnit4ClassRunner`对Spring应用程序进行集成测试__

>原书P77~78代码

>文字

>Loads application context：加载应用程序上下文

>Injects address service：注入地址服务

>Tests address service：测试地址服务

如你所见，`AddressServiceTests`上加注了`@RunWith`和`@ContextConfiguration`注解。`@RunWith`的参数是`SpringJUnit4ClassRunner.class`，开启了Spring集成测试支持。***{![在Spring 4.2里，你可以选择基于规则的`SpringClassRule`和`SpringMethodRule`来代替`SpringJUnit4ClassRunner`。]}***与此同时，`@ContextConfiguration`指定了如何加载应用程序上下文。此处我们让它加载`AddressBookConfiguration`里配置的Spring应用程序上下文。

除了加载应用程序上下文，`SpringJUnit4ClassRunner`还能通过自动织入从应用程序上下文里向测试本身注入Bean。因为这是一个针对`AddressService` Bean的测试，所以需要将它注入测试。最后，`testService()`方法调用地址服务并验证了结果。

虽然`@ContextConfiguration`在加载Spring应用程序上下文的过程中做了很多事情，但它没能加载完整的Spring Boot。Spring Boot应用程序最终是由`SpringApplication`加载的。它可以显式加载（如代码清单2-1所示），在这里也可以使用`SpringBootServletInitializer`（我们会在第8章里看到具体做法）。`SpringApplication`不仅加载应用程序上下文，还会开启日志、加载外部属性（application.properties或application.yml），以及其他Spring Boot特性。用`@ContextConfiguration`则得不到这些特性。

要在集成测试里获得这些特性，可以把`@ContextConfiguration`替换为Spring Boot的`@SpringApplicationConfiguration`：
```
@RunWith(SpringJUnit4ClassRunner.class)
@SpringApplicationConfiguration(
      classes=AddressBookConfiguration.class)
public class AddressServiceTests {
  ...
}
```
`@SpringApplicationConfiguration`的用法和`@ContextConfiguration`大致相同，但也有不同的地方，`@SpringApplicationConfiguration`加载Spring应用程序上下文的方式同`SpringApplication`相同，处理方式和生产应用程序中的情况相同。这包括加载外部属性和Spring Boot日志。

我们有充分的理由说，在大多数情况下，为Spring Boot应用程序编写测试时应该用`@SpringApplicationConfiguration`代替`@ContextConfiguration`。在本章中，我们当然也会用`@SpringApplicationConfiguration`来为Spring Boot应用程序（包括那些面向前端的应用程序）编写测试。

说到Web测试，这正是我们接下来要做的。

| [PREV](https://github.com/5202m/spring-boot-in-action-zh-cn/blob/master/03WallsCh03-3.3.md) | [NEXT](https://github.com/5202m/spring-boot-in-action-zh-cn/blob/master/04WallsCh04-4.2.md) |
