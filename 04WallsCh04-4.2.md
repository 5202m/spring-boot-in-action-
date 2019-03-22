## 4.2 测试Web应用程序

Spring MVC有一个优点：它的编程模型是围绕POJO展开的，在POJO上添加注解，声明如何处理Web请求。这种编程模型不仅简单，还让你能像对待应用程序中的其他组件一样对待这些控制器。你还可以针对这些控制器编写测试，就像测试POJO一样。

举例来说，考虑`ReadingListController`里的`addToReadingList()`方法：
```
@RequestMapping(method=RequestMethod.POST)
public String addToReadingList(Book book) {
  book.setReader(reader);
  readingListRepository.save(book);
  return "redirect:/readingList";
}
```
如果忽略`@RequestMapping`注解，你得到的就是一个相当基础的Java方法。你立马就能想到这样一个测试，提供一个`ReadingListRepository`的模拟实现，直接调用`addToReadingList()`，判断返回值并验证对`ReadingListRepository`的`save()`方法有过调用。

该测试的问题在于，它仅仅测试了方法本身，当然，这要比没有测试好一点。然而，它没有测试该方法处理/readingList的`POST`请求的情况，也没有测试表单域绑定到`Book`参数的情况。虽然你可以判断返回的`String`包含特定值，但没法明确测试请求在方法处理完之后是否真的会重定向到/readingList。

要恰当地测试一个Web应用程序，你需要投入一些实际的HTTP请求，确认它能正确地处理那些请求。幸运的是，Spring Boot开发者有两个可选的方案能实现这类测试。

- Spring Mock MVC：能在一个近似真实的模拟Servlet容器里测试控制器，而不用实际启动应用服务器。
- Web integration tests：在嵌入式Servlet容器（比如Tomcat或Jetty）里启动应用程序，在真正的应用服务器里执行测试。

这两种方法各有利弊。很明显，启动一个应用服务器会比模拟Servlet容器要慢一些，但毫无疑问基于服务器的测试会更接近真实环境，更接近部署到生产环境运行的情况。

接下来，你会看到如何使用Spring Mock MVC测试框架来测试Web应用程序。然后，在4.3节里你会看到如何为运行在应用服务器里的应用程序编写测试。

### 4.2.1 模拟Spring MVC

早在Spring 3.2，Spring Framework就有了一套非常实用的Web应用程序测试工具，能模拟Spring MVC，不需要真实的Servlet容器也能对控制器发送HTTP请求。Spring的Mock MVC框架模拟了Spring MVC的很多功能。它几乎和运行在Servlet容器里的应用程序一样，尽管实际并非如此。

要在测试里设置Mock MVC，可以使用`MockMvcBuilders`，该类提供了两个静态方法。

- `standaloneSetup()`：构建一个Mock MVC，提供一个或多个手工创建并配置的控制器。
- `webAppContextSetup()`：使用Spring应用程序上下文来构建Mock MVC，该上下文里可以包含一个或多个配置好的控制器。

两者的主要区别在于，`standaloneSetup()`希望你手工初始化并注入你要测试的控制器，而`webAppContextSetup()`则基于一个`WebApplicationContext`的实例，通常由Spring加载。前者同单元测试更加接近，你可能只想让它专注于单一控制器的测试，而后者让Spring加载控制器及其依赖，以便进行完整的集成测试。

我们要用的是`webAppContextSetup()`。Spring完成了`ReadingListController`的初始化，并从Spring Boot自动配置的应用程序上下文里将其注入，我们直接对其进行测试。

`webAppContextSetup()`接受一个`WebApplicationContext`参数。因此，我们需要为测试类加上`@WebAppConfiguration`注解，使用`@Autowired`将`WebApplicationContext`作为实例变量注入测试类。代码清单4-2演示了Mock MVC测试的执行入口。

__代码清单4-2 为集成测试控制器创建Mock MVC__

>原文P80~81代码

>**文字**

>Enables web context testing  开启Web上下文测试

>Injects WebApplicationContext：注入`WebApplicationContext`

>Sets up MockMvc：设置`MockMvc`

`@WebAppConfiguration`注解声明，由`SpringJUnit4ClassRunner`创建的应用程序上下文应该是一个`WebApplicationContext`（相对于基本的非Web`ApplicationContext`）。

`setupMockMvc()`方法上添加了JUnit的`@Before`注解，表明它应该在测试方法之前执行。它将`WebApplicationContext`注入`webAppContextSetup()`方法，然后调用`build()`产生了一个`MockMvc`实例，该实例赋给了一个实例变量，供测试方法使用。

现在我们有了一个`MockMvc`，已经可以开始写测试方法了。我们先写个简单的测试方法，向/readingList发送一个HTTP `GET`请求，判断模型和视图是否满足我们的期望。下面的`homePage()`测试方法就是我们所需要的：

```
@Test
public void homePage() throws Exception {
  mockMvc.perform(MockMvcRequestBuilders.get("/readingList"))
          .andExpect(MockMvcResultMatchers.status().isOk())
          .andExpect(MockMvcResultMatchers.view().name("readingList"))
          .andExpect(MockMvcResultMatchers.model().attributeExists("books"))
          .andExpect(MockMvcResultMatchers.model().attribute("books",
                   Matchers.is(Matchers.empty())));
}
```

如你所见，我们在这个测试方法里使用了很多静态方法，包括Spring的`MockMvcRequestBuilders`和`MockMvcResultMatchers`里的静态方法，还有Hamcrest库的`Matchers`里的静态方法。在深入探讨这个测试方法前，先添加一些静态`import`，这样代码看起来更清爽一些：

```
import static org.hamcrest.Matchers.*;
import static org.springframework.test.web.servlet.request.
        ➥ MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.
        ➥ MockMvcResultMatchers.*;
```
有了这些静态`import`后，测试方法可以稍作调整：
```
@Test
public void homePage() throws Exception {
mockMvc.perform(get("/readingList"))
        .andExpect(status().isOk())
        .andExpect(view().name("readingList"))
        .andExpect(model().attributeExists("books"))
        .andExpect(model().attribute("books", is(empty())));
}
```
现在这个测试方法读起来就很自然了。首先向/readingList发起一个`GET`请求，接下来希望该请求处理成功（`isOk()`会判断HTTP 200响应码），并且视图的逻辑名称为`readingList`。测试还要断定模型包含一个名为`books`的属性，该属性是一个空集合。所有的断言都很直观。

值得一提的是，此处完全不需要将应用程序部署到Web服务器上，它是运行在模拟的Spring MVC中的，刚好能通过`MockMvc`实例处理我们给它的HTTP请求。

太酷了，不是吗？

让我们再来看一个测试方法，这次会更有趣，我们实际发送一个HTTP `POST`请求提交一本新书。我们应该期待`POST`请求处理后重定向回/readingList，模型将包含新添加的图书。代码清单4-3演示了如何通过Spring的Mock MVC来实现这个测试。

__代码清单4-3 测试提交一本新书__

>原书P82~83代码

>**文字**

>Performs POST request：执行POST请求

>Sets up expected book：配置期望的图书

>Performs GET request：执行GET请求

很明显，代码清单4-3里的测试更加复杂，实际上是两个测试放在一个方法里。第一部分提交图书并检查了请求的结果，第二部分执行了一次对主页的`GET`请求，检查新建的图书是否在模型中。

在提交图书时，我们必须确保内容类型（通过`MediaType.APPLICATION_FORM_URLENCODED`）设置为application/x-www-form-urlencoded，这才是运行应用程序时浏览器会发送的内容类型。随后，要用`MockMvcRequestBuilders`的`param`方法设置表单域，模拟要提交的表单。一旦请求执行，我们要检查响应是否是一个到/readingList的重定向。

假定以上测试都通过，我们进入第二部分。首先设置一个`Book`对象，包含想要的值。我们用这个对象和首页获取的模型的值进行对比。

随后要对/readingList发起一个`GET`请求，大部分内容和我们之前测试主页时一样，只是之前模型中有一个空集合，而现在有一个集合项。这里要检查它的内容是否和我们创建的`expectedBook`一致。如此一来，我们的控制器看来保存了发送给它的图书，完成了工作。

至此，这些测试验证了一个未经保护的应用程序，和我们在第2章里写的应用程序很类似。但如果我们想要测试一个安全加固过的应用程序（比如我们在第3章里写的程序），又该怎么办？

### 4.2.2 测试Web安全

Spring Security能让你非常方便地测试安全加固后的Web应用程序。为了利用这点优势，你必须在项目里添加Spring Security的测试模块。要在Gradle里做到这一点，你需要的就是以下`testCompile`依赖：

```
testCompile("org.springframework.security:spring-security-test")
```

如果你用的是Maven，则添加以下`<dependency>`：

```
<dependency>
  <groupId>org.springframework.security</groupId>
  <artifactId>spring-security-test</artifactId>
  <scope>test</scope>
</dependency>
```

应用程序的Classpath里有了Spring Security的测试模块之后，只需在创建`MockMvc`实例时运用Spring Security的配置器。
```
@Before
public void setupMockMvc() {
  mockMvc = MockMvcBuilders
    .webAppContextSetup(webContext)
    .apply(springSecurity())
    .build();
}
```
`springSecurity()`方法返回了一个Mock MVC配置器，为Mock MVC开启了Spring Security支持。只需像上面这样运用就行了，Spring Security会介入`MockMvc`上执行的每个请求。具体的安全配置取决于你如何配置Spring Security（或者Spring Boot如何自动配置Spring Security）。在阅读列表这个应用程序里，我们在第3章里创建`SecurityConfig.java`时，配置也是如此。

>__`springSecurity()`方法__

>`springSecurity()`是`SecurityMockMvcConfigurers`的一个静态方法，考虑到可读性，我已经将其静态导入。

开启了Spring Security之后，在请求主页的时候，我们便不能只期待HTTP 200响应。如果请求未经身份验证，我们应该期待重定向到登录页面：
```
@Test
public void homePage_unauthenticatedUser() throws Exception {
  mockMvc.perform(get("/"))
    .andExpect(status().is3xxRedirection())
    .andExpect(header().string("Location",
                               "http://localhost/login"));
}
```
但是我们又该如何发起一个经过身份验证的请求呢？Spring Security提供了两个注解。

- `@WithMockUser`：加载安全上下文，其中包含一个`UserDetails`，使用了给定的用户名、密码和授权。
- `@WithUserDetails`：根据给定的用户名查找`UserDetails`对象，加载安全上下文。

在这两种情况下，Spring Security的安全上下文都会加载一个`UserDetails`对象，添加了该注解的测试方法在运行过程中都会使用该对象。`@WithMockUser`注解是两者里比较基础的那个，允许显式声明一个`UserDetails`，并加载到安全上下文：
```
@Test
@WithMockUser(username="craig",
              password="password",
              roles="READER")
public void homePage_authenticatedUser() throws Exception {
  ...
}
```
如你所见，`@WithMockUser`绕过了对`UserDetails`对象的正常查询，用给定的值创建了一个`UserDetails`对象取而代之。在简单的测试里，这就够用了。但我们的测试需要`Reader`（实现了`UserDetails`）而非`@WithMockUser`创建的通用`UserDetails`。为此，我们需要`@WithUserDetails`。

`@WithUserDetails`注解使用事先配置好的`UserDetailsService`来加载`UserDetails`对象。回想一下第3章，我们配置了一个`UserDetailsService` Bean，它会根据给定的用户名查找并返回一个`Reader`对象。太完美了！所以我们要为测试方法添加`@WithUserDetails`注解，如代码清单4-4所示。

__代码清单4-4 测试带有用户身份验证的安全加固方法__

>原书P85代码

>**文字**

>Uses “craig” user ：使用craig用户

>Sets up expected Reader：配置期望的`Reader`

>Performs GET request：发起`GET`请求

在代码清单4-4里，我们通过`@WithUserDetails`注解声明要在测试方法执行过程中向安全上下文里加载craig用户。`Reader`会放入模型，该测试方法先创建了一个期望的`Reader`对象，后续可以用来进行比较。随后`GET`请求发起，也有了针对视图名和模型内容的断言，其中包括名为`reader`的模型属性。

同样，此处没有启动Servlet容器来运行这些测试，Spring的Mock MVC取代了实际的Servlet容器。这样做的好处是测试方法运行相对较快。因为不需要等待服务器启动，而且不需要打开Web浏览器发送表单，所以测试比较简单快捷。

不过，这并不是一个完整的测试。它比直接调用控制器方法要好，但它并没有真的在Web浏览器里执行应用程序，验证呈现出的视图。为此，我们需要启动一个真正的Web服务器，用真实浏览器来访问它。让我们来看看Spring Boot如何启动一个真实的Web服务器来帮助测试。
