## 7.5 保护Actuator端点

很多Actuator端点发布的信息都可能涉及敏感数据，还有一些端点，（比如/shutdown）非常危险，可以用来关闭应用程序。因此，保护这些端点尤为重要，能访问它们的只能是那些经过授权的客户端。

实际上，Actuator的端点保护可以用和其他URL路径一样的方式——使用Spring Security。在Spring Boot应用程序中，这意味着将Security起步依赖作为构建依赖加入，然后让安全相关的自动配置来保护应用程序，其中当然也包括了Actuator端点。

在第3章，我们看到了默认安全自动配置如何把所有URL路径保护起来，要求HTTP基本身份验证，用户名是user，密码在启动时随机生成并写到日志文件里去。这不是我们所希望的Actuator保护方式。

我们已经添加了一些自定义安全配置，仅限带有READER权限的授权用访问根URL路径（`/`）。要保护Actuator的端点，我们需要对`SecurityConfig.java`的`configure()`方法做些修改。

举例来说，你想要保护/shutdown端点，仅允许拥有ADMIN权限的用户访问，代码清单7-13就是新的`configure()`方法。

__代码清单7-13 保护/shutdown端点__

```
@Override
protected void configure(HttpSecurity http) throws Exception {
  http
    .authorizeRequests()
      .antMatchers("/").access("hasRole('READER')")
      .antMatchers("/shutdown").access("hasRole('ADMIN')")
      .antMatchers("/**").permitAll()
    .and()
    .formLogin()
      .loginPage("/login")
      .failureUrl("/login?error=true");
}
```
>文字

>Require ADMIN access：要求有ADMIN权限

现在要访问/shutdown端点，必须用一个带ADMIN权限的用户来做身份验证。

然而，第3章里的自定义`UserDetailsService`只对通过`ReaderRepository`加载的用户赋予READER权限。因此，你需要创建一个更聪明的`UserDetailsService`实现，对某些用户赋予ADMIN权限。你可以配置一个额外的身份验证实现，比如代码清单7-14里的内存实现。

__代码清单7-14 添加一个内存里的admin用户__

```
@Override
protected void configure(
            AuthenticationManagerBuilder auth) throws Exception {
  auth
    .userDetailsService(new UserDetailsService() {
      @Override
      public UserDetails loadUserByUsername(String username)
          throws UsernameNotFoundException {
        UserDetails user = readerRepository.findOne(username);
        if (user != null) {
          return user;
        }
        throw new UsernameNotFoundException(
                      "User '" + username + "' not found.");
      }
    })
    .and()
    .inMemoryAuthentication()
      .withUser("admin").password("s3cr3t")
                        .roles("ADMIN", "READER");
}
```
>文字

>Reader authentication：Reader身份验证

>Admin authentication：Admin身份验证

新加的内存身份验证中，用户名定义为admin，密码为s3cr3t，同时被授予ADMIN和READER权限。

现在，除了那些拥有ADMIN权限的用户，谁都无法访问/shutdown端点。但Actuator的其他端点呢？假设你只想让ADMIN的用户访问它们（像/shutdown一样），可以在调用`antMatchers()`时列出这些URL。例如，要保护/metrics、/confiprops和/shutdown，可以像这样调用`antMatchers()`：

```
.antMatchers("/shutdown", "/metrics", "/configprops")
                          .access("hasRole('ADMIN')")
```

虽然这么做能奏效，但也只适用于少数Actuator端点的保护。如果要保护全部Actuator端点，这种做法就不太方便了。

比起在调用`antMatchers()`方法时显式地列出所有的Actuator端点，用通配符在一个简单的Ant风格表达式里匹配全部的Actuator端点更容易。但是，这么做也小有点挑战，因为不同的端点路径之间没有什么共同点，我们也不能在`/**`上运用ADMIN权限。这样一来，除了根路径（`/`）之外，什么要有ADMIN权限。

为此，可以通过`management.context-path`属性设置端点的上下文路径。默认情况下，这个属性是空的，所以Actuator的端点路径都是相对于根路径的。在application.yaml里增加如下内容，可以让这些端点都带上`/mgmt`前缀。

```
management:
  context-path: /mgmt
```

你也可以在application.properties里做类似的事情：

```
management.context-path=/mgmt
```

将`management.context-path`设置为/mgmt后，所有的Actuator端点都会与/mgmt路径相关。例如，/metrics端点的URL会变为/mgmt/metrics。

有了这个新的路径，我们就有了公共的前缀，在为Actuator端点赋予ADMIN权限限制时就能借助这个公共前缀：

```
.antMatchers("/mgmt/**").access("hasRole('ADMIN')")
```

现在所有以/mgmt开头的请求（包含了所有的Actuator端点），都只让授予了ADMIN权限的认证用户访问。

## 7.6 小结

想弄清楚运行的应用程序里正在发生什么，这是件很困难的事。Spring Boot的Actuator为你打开了一扇大门，深入Spring Boot应用程序的内部细节。它发布的组件、度量和指标能帮你理解应用程序的运作情况。

在本章，我们先了解了Actuator的Web端点——通过HTTP发布运行时细节信息的REST端点。这些端点的功能包括查看Spring应用程序上下文里所有的Bean、查看自动配置决策、查看Spring MVC映射、查看线程活动、查看应用程序健康信息，还有多种度量、指标和计数器。

除了Web端点，Actuator还提供了另外两种获取它所提供信息的途径。远程shell让你能在shell里安全地连上应用程序，发起指令，获得与Actuator端点相同的数据。与此同时，所有的Actuator端点也都发布成了MBean，可以通过JMX客户端进行监控和管理。

随后我们还了解了如何定制Actuator，包括如何通过端点的ID来修改Actuator端点的路径，如何启用和禁用端点，诸如此类。我们还插入了一些定制的度量信息，创建了定制的跟踪信息仓库，替换了默认的内存跟踪仓库。

最后，我们学习了如何保护Actuator的端点，只让经过授权的用户访问它们。

接下来，在第8章里，我们将看到如何让应用程序从编码阶段过渡到生产阶段，了解Spring Boot如何协助我们在多种不同的平台上进行部署，包括传统的应用容器和云平台。
