# 第3章 自定义配置

> 本章内容

>- 覆盖自动配置的Bean
- 用外置属性进行配置
- 自定义错误页

能自由选择真是太棒了。如果你订过比萨（有没订过的吗？）就会知道，你完全可以掌控薄饼上放哪些辅料。选定腊肠、意大利辣香肠、青辣椒和额外芝士的时候，你就是在按照自己的要求配置比萨。

另一方面，大部分比萨店也提供某种形式的自动配置。你可以点荤比萨、素比萨、香辣意大利比萨，或者是自动配置比萨中的极品——至尊比萨。在下单时，你并没有指定具体的辅料，你所点的比萨种类决定了所用的辅料。

但如果你想要至尊比萨上的全部辅料，还想要加墨西哥胡椒，又不想放蘑菇该怎么办？你偏爱辣食又不喜欢吃菌类，自动配置不适合你的口味，你就只能自己配置比萨了吗？当然不是，大部分比萨店会让你以菜单上已有的选项为基础进行定制。

使用传统Spring配置的过程，就如同订比萨的时候自己指定全部的辅料。你可以完全掌控Spring配置的内容，可是显式声明应用程序里全部的Bean并不是明智之举。而Spring Boot自动配置就像是从菜单上选一份特色比萨，让Spring Boot处理各种细节比自己声明上下文里全部的Bean要容易很多。

幸运的是，Spring Boot自动配置非常灵活。就像比萨厨师可以不在你的比萨里放蘑菇，而是加墨西哥胡椒一样，Spring Boot能让你参与进来，影响自动配置的实施。

本章我们将看到两种影响自动配置的方式——使用显式配置进行覆盖和使用属性进行精细化配置。我们还会看到如何使用Spring Boot提供的钩子引入自定义的错误页。

## 3.1 覆盖Spring Boot自动配置

一般来说，如果不用配置就能得到和显式配置一样的结果，那么不写配置是最直接的选择。既然如此，那干嘛还要多做额外的工作呢？如果不用编写和维护额外的配置代码也行，那何必还要它们呢？

大多数情况下，自动配置的Bean刚好能满足你的需要，不需要去覆盖它们。但某些情况下，Spring Boot在自动配置时还不能很好地进行推断。

这里有个不错的例子：当你在应用程序里添加安全特性时，自动配置做得还不够好。安全配置并不是放之四海而皆准的，围绕应用程序安全有很多决策要做，Spring Boot不能替你做决定。虽然Spring Boot为安全提供了一些基本的自动配置，但是你还是需要自己覆盖一些配置以满足特定的安全要求。

想知道如何用显式的配置来覆盖自动配置，我们先从为阅读列表应用程序添加Spring Security入手。在了解自动配置提供了什么之后，我们再来覆盖基础的安全配置，以满足特定的场景需求。

### 3.1.1 保护应用程序

Spring Boot自动配置让应用程序的安全工作变得易如反掌，你要做的只是添加Security起步依赖。以Gradle为例，应添加如下依赖：
```
compile("org.springframework.boot:spring-boot-starter-security")
```
如果使用Maven，那么你要在项目的`<dependencies>`块中加入如下`<dependency>`：
```
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

这样就搞定了！重新构建应用程序后运行即可，现在这就是一个安全的Web应用程序了！Security起步依赖在应用程序的Classpath里添加了Spring Secuirty（和其他一些东西）。Classpath里有Spring Security后，自动配置就能介入其中创建一个基本的Spring Security配置。

试着在浏览器里打开该应用程序，你马上就会看到HTTP基础身份验证对话框。此处的用户名是user，密码就有点麻烦了。密码是在应用程序每次运行时随机生成后写入日志的，你需要查找日志消息（默认写入标准输出），找到此类内容：

```
Using default security password: d9d8abe5-42b5-4f20-a32a-76ee3df658d9
```

我不能肯定，但我猜这个特定的安全配置并不是你的理想选择。首先，HTTP基础身份验证对话框有点粗糙，对用户并不友好。而且，我敢打赌你一般不会开发这种只有一个用户的应用程序，而且他还要从日志文件里找到自己的密码。因此，你会希望修改Spring Security的一些配置，至少要有一个好看一些的登录页，还要有一个基于数据库或LDAP（Lightweight Directory Access Protocol，轻量级目录访问协议）用户存储的身份验证服务。

让我们看看如何写出Spring Secuirty配置，覆盖自动配置的安全设置吧。

### 3.1.2 创建自定义的安全配置

覆盖自动配置很简单，就当自动配置不存在，直接显式地写一段配置。这段显式配置的形式不限，Spring支持的XML和Groovy形式配置都可以。

在编写显式配置时，我们会专注于Java形式的配置。在Spring Security的场景下，这意味着写一个扩展了`WebSecurityConfigurerAdapter`的配置类。代码清单3-1中的`SecurityConfig`就是我们需要的东西。

__代码清单3-1 覆盖自动配置的显式安全配置__

> 原书P51~52代码

>**文字**

>Require READER access ：要求登录者有READER角色

>Set login form path ：设置登录表单的路径

>Define custom UserDetailsService ：定义自定义`UserDetailsService`

`SecurityConfig`是个非常基础的Spring Security配置，尽管如此，它还是完成了不少安全定制工作。通过这个自定义的安全配置类，我们让Spring Boot跳过了安全自动配置，转而使用我们的安全配置。

扩展了`WebSecurityConfigurerAdapter`的配置类可以覆盖两个不同的`configure()`方法。在`SecurityConfig`里，第一个`configure()`方法指明，“/”（`ReadingListController`的方法映射到了该路径）的请求只有经过身份认证且拥有READER角色的用户才能访问。其他的所有请求路径向所有用户开放了访问权限。这里还将登录页和登录失败页（带有一个`error`属性）指定到了/login。

Spring Security为身份认证提供了众多选项，后端可以是JDBC（Java Database Connectivity，Java数据库连接）、LDAP和内存用户存储。在这个应用程序中，我们会通过JPA用数据库来存储用户信息。第二个`configure()`方法设置了一个自定义的`UserDetailsService`，这个服务可以是任意实现了`UserDetailsService`的类，用于查找指定用户名的用户。代码清单3-2提供了一个匿名内部类实现，简单地调用了注入`ReaderRepository`（这是一个Spring Data JPA仓库接口）的`findOne()`方法。

__代码清单3-2 用来持久化读者信息的仓库接口__

>原书P53代码

>**文字**

>Persist readers via JPA：  通过JPA持久化读者

和`BookRepository`类似，你无需自己实现`ReaderRepository`。这是因为它扩展了`JpaRepository`，Spring Data JPA会在运行时自动创建它的实现。这为你提供了18个操作`Reader`实体的方法。

说到`Reader`实体，`Reader`类（如代码清单3-3所示）就是最后一块拼图了，它就是一个简单的JPA实体，其中有几个字段用来存储用户名、密码和用户全名。

__代码清单3-3 定义`Reader`的JPA实体__

>原书P53~54

>**文字**

>Reader fields ：Reader字段

>Grant READER privilege ：授予READER权限

>Do not expire, lock, or disable：不过期，不加锁，不禁用

如你所见，`Reader`用了`@Entity`注解，所以这是一个JPA实体。此外，它的`username`字段上有`@Id`注解，表明这是实体的ID。这个选择无可厚非，因为`username`应该能唯一标识一个`Reader`。

你应该还注意到`Reader`实现了`UserDetails`接口以及其中的方法，这样`Reader`就能代表Spring Security里的用户了。`getAuthorities()`方法被覆盖过了，始终会为用户授予READER权限。`isAccountNonExpired()`、 `isAccountNonLocked()`、`isCredentialsNonExpired()`和`isEnabled()`方法都返回`true`，这样读者账户就不会过期，不会被锁定，也不会被撤销。

重新构建并重启应用程序后，你应该就能以读者身份登录应用程序了。

> __保持简单__

> 在一个大型应用程序里，赋予用户的授权本身也可能是实体，它们被维护在独立的数据表里。同样，表示一个账户是否为非过期、非锁定且可用的布尔值也是数据库里的字段。但是，出于演示考虑，我决定让这些细节保持简单，以免分散我们的注意力，影响正在讨论的话题——我说的是覆盖Spring Boot自动配置。

在安全配置方面，我们还能做更多事情***{![想要深入了解Spring Security，可以参考《Spring实战（第4版）》中的第9章和第14章。]}***，但此刻这样就足够了，上面的例子足以演示如何覆盖Spring Boot提供的安全自动配置。

再重申一次，想要覆盖Spring Boot的自动配置，你所要做的仅仅是编写一个显式的配置。Spring Boot会发现你的配置，随后降低自动配置的优先级，以你的配置为准。想弄明白这是如何实现的，让我们揭开Spring Boot自动配置的神秘面纱，看看它是如何运作的，以及它是怎么允许自己被覆盖的。

### 3.1.3 掀开自动配置的神秘面纱

正如我们在2.3.3节里讨论的那样，Spring Boot自动配置自带了很多配置类，每一个都能运用在你的应用程序里。它们都使用了Spring 4.0的条件化配置，可以在运行时判断这个配置是该被运用，还是该被忽略。

大部分情况下，表2-1里的`@ConditionalOnMissingBean`注解是覆盖自动配置的关键。Spring Boot的`DataSourceAutoConfiguration`中定义的`JdbcTemplate` Bean就是一个非常简单的例子，演示了`@ConditionalOnMissingBean`如何工作：
```
@Bean
@ConditionalOnMissingBean(JdbcOperations.class)
public JdbcTemplate jdbcTemplate() {
  return new JdbcTemplate(this.dataSource);
}
```
`jdbcTemplate()`方法上添加了`@Bean`注解，在需要时可以配置出一个`JdbcTemplate` Bean。但它上面还加了`@ConditionalOnMissingBean`注解，要求当前不存在`JdbcOperations`类型（`JdbcTemplate`实现了该接口）的Bean时才生效。如果当前已经有一个`JdbcOperations` Bean了，条件即不满足，不会执行`jdbcTemplate()`方法。

什么情况下会存在一个`JdbcOperations` Bean呢？Spring Boot的设计是加载应用级配置，随后再考虑自动配置类。因此，如果你已经配置了一个`JdbcTemplate` Bean，那么在执行自动配置时就已经存在一个`JdbcOperations`类型的Bean了，于是忽略自动配置的`JdbcTemplate` Bean。

关于Spring Security，自动配置会考虑几个配置类。在这里讨论每个配置类的细节是不切实际的，但覆盖Spring Boot自动配置的安全配置时，最重要的一个类是`SpringBootWebSecurityConfiguration`。以下是其中的一个代码片段：
```
@Configuration
@EnableConfigurationProperties
@ConditionalOnClass({ EnableWebSecurity.class })
@ConditionalOnMissingBean(WebSecurityConfiguration.class)
@ConditionalOnWebApplication
public class SpringBootWebSecurityConfiguration {

  ...

}
```
如你所见，`SpringBootWebSecurityConfiguration`上加了好几个注解。看到`@ConditionalOnClass`注解后，你就应该知道Classpath里必须要有`@EnableWebSecurity`注解。`@ConditionalOnWebApplication`说明这必须是个Web应用程序。`@ConditionalOnMissingBean`注解才是我们的安全配置类代替`SpringBootWebSecurityConfiguration`的关键所在。

`@ConditionalOnMissingBean`注解要求当下没有`WebSecurityConfiguration`类型的Bean。虽然表面上我们并没有这么一个Bean，但通过在`SecurityConfig`上添加`@EnableWebSecurity`注解，我们实际上间接创建了一个`WebSecurityConfiguration` Bean。所以在自动配置时，这个Bean就已经存在了，`@ConditionalOnMissingBean`条件不成立，`SpringBootWebSecurityConfiguration`提供的配置就被跳过了。

虽然Spring Boot的自动配置和`@ConditionalOnMissingBean`让你能显式地覆盖那些可以自动配置的Bean，但并不是每次都要做到这种程度。让我们来看看怎么通过设置几个简单的配置属性调整自动配置组件吧。
