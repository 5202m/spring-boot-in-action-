## 5.3 用CLI运行测试

测试是软件项目的重要组成部分，Spring Boot CLI当然没有忽略测试。因为基于CLI的应用程序并未涉及传统的构建系统，所以CLI提供了一个`test`命令来运行测试。

在试验`test`命令前，你先要写一个测试。测试可以放在项目中的任何位置。我建议将其与主要组件分开放置，最好放在一个子目录里。这个子目录的名字随意。我在这里将其命名为tests：
```
$ mkdir tests
```
在tests目录里，创建一个名为ReadingListControllerTest.groovy的新Groovy脚本，编写针对`ReadingListController`的测试。代码清单5-3是个简单的测试，测试控制器能否正确处理HTTP `GET`请求。

__代码清单5-3 `ReadingListController`的Groovy测试__

> 原书P103代码

>**文字**

>Mock ReadingListRepository：模拟`ReadingListRepository`

>Perform and test GET request：执行并测试`GET`请求

如你所见，这就是个简单的JUnit测试，使用了Spring的模拟MVC测试支持功能，对控制器发起`GET`请求。最先设置的是`ReadingListRepository`的一个模拟实现，它会返回一个包含单一`Book`项的列表。随后，测试创建了一个`ReadingListController`实例，将模拟仓库注入`readingListRepository`属性。最后，配置了一个`MockMvc`对象，发起`GET`请求，对期望的视图名称和模型内容进行断言。

但是，此处运行测试要比说明测试更重要。使用CLI的`test`命令，可以像下面这样在命令行里执行测试：
```
$ spring test tests/ReadingListControllerTest.groovy
```
本例中，我明确选中了`ReadingListControllerTest`作为要运行的测试。如果tests/目录里有多个测试，你想要全部运行，可以在`test`命令中指定目录名：
```
$ spring test tests
```
如果你倾向于编写Spock说明而非JUnit测试，那么你一定会很高兴，因为CLI的`test`命令也可以运行Spock说明，代码清单5-4的`ReadingListControllerSpec`就演示了这一功能。

__代码清单5-4 测试`ReadingListController`的Spock说明__

>原文P104代码

>**文字**

>Mock ReadingListRepository：模拟的`ReadingListRepository`

>Perform GET request：执行`GET`请求

>Test results：测试结果

`ReadingListControllerSpec`只是简单地把`ReadingListControllerTest`从JUnit测试翻译成了Spock说明。如你所见，它只是直白地表述了这么一个过程。对“/”出现`GET`请求时，响应中应该包含名为readingList的视图。模型里的`books`键所对应的就是期待的图书列表。

Spock说明也可以通过`spring test tests`来运行`ReadingListControllerSpec`。运行方式和基于JUnit的测试如出一辙。

一旦写好代码，通过了全部测试，你就该部署项目了。让我们来看看Spring Boot CLI是如何帮助产生一个可部署的产物的。
