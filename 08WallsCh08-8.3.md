## 8.3 推上云端

服务器硬件的购买和维护成本很高。大流量很难通过适当扩展服务器去处理，这种做法在某些组织中甚至是禁忌。现如今，相比在自己的数据中心运行应用程序，把它们部署到云上是更引人注目，而且划算的做法。

目前有多个云平台可供选择，而那些提供Platform as a Service（PaaS）能力的平台无疑是最有吸引力的。PaaS提供了现成的应用程序部署平台，带有附加服务（比如数据库和消息代理），可以绑定到应用程序上。除此之外，当你的应用程序要求提供更大的马力时，云平台能轻松实现应用程序在运行时向上（或向下）伸缩，只需添加或删除实例即可。

之前我们已经把阅读列表应用程序部署到了传统的应用服务器上，现在再试试将其部署到云上。我们将把应用程序部署到Cloud Foundry和Heroku这两个著名的PaaS平台上。

### 8.3.1 部署到Cloud Foundry

Cloud Foundry是Pivotal的PaaS平台。这家公司也赞助了Spring Framework和Spring平台里的其他库。Cloud Foundry里最吸引人的特点之一就是它既有开源版本，也有多个商业版本。你可以选择在何处运行Cloud Foundry。它甚至还可以在公司数据中心的防火墙后运行，提供私有云。

我打算将阅读列表应用程序部署到Pivotal Web Services（PWS）上。这是一个由Pivotal托管的公共Cloud Foundry，地址是[http://run.pivotal.io](http://run.pivotal.io)。如果想使用PWS，你可以注册一个账号。PWS提供为期60天的免费试用，在试用期间无需提交任何信用卡信息。

在注册了PWS后，可以从[https://console.run.pivotal.io/tools](https://console.run.pivotal.io/tools)下载并安装`cf`命令行工具。你可以通过这个工具将应用程序推上Cloud Foundry。但你要先用这个工具登录自己的PWS账号。

```
$ cf login -a https://api.run.pivotal.io
API endpoint: https://api.run.pivotal.io

Email> {your email}

Password> {your password}
Authenticating...
OK
```
现在我们已经可以把阅读列表应用程序传到云上了。实际上，我们的项目已经做好了部署到Cloud Foundry的准备，只需使用`cf push`命令把它推上去就好。
```
$ cf push sbia-readinglist -p build/libs/readinglist.war
```
`cf push`命令的第一个参数指定了应用程序在Cloud Foundry里的名称。这个名称将被用作托管应用程序的子域名。本例中，应用程序的完整域名将是[http://sbia-readinglist.cfapps.io](http://sbia-readinglist.cfapps.io)。因此，应用程序的命名很重要。名字必须独一无二，这样才不会和Cloud Foundry里部署的其他应用程序（包括其他用户部署的应用程序）发生冲突。

因为空想一个独一无二的名称有点困难，所以`cf push`命令提供了一个`--random-route`选项，可以为你随机产生一个子域名。下面的例子演示了如何上传阅读列表应用程序，生成一个随机的子域名。

```
$ cf push sbia-readinglist -p build/libs/readinglist.war --random-route
```

在使用了`--random-route`后，还是要设定应用程序名称。会有两个随机选择的单词添加到后面，组成子域名。（在我自己尝试的时候，生成的子域名是sbia-readinglist-gastroenterological-stethoscope。）

> __不仅是WAR文件__

>虽然我们部署的应用程序是一个WAR文件，但Cloud Foundry也可以部署其他格式的Spring Boot应用程序，包括可执行的JAR文件，甚至Spring Boot CLI开发的未经编译的Groovy脚本。

如果一切顺利，我们部署的应用程序应该可以处理请求。假设子域名是sbia-readinglist，你可以用浏览器访问[http://sbia-readinglist.cfapps.io](http://sbia-readinglist.cfapps.io)，看看效果。你应该会被引导到登录页。回想一下，数据库迁移脚本中插入了一个名为craig的用户，密码是password，可以以此登录应用程序。

你可以在应用程序里随便点点，加几本书。所有的东西都可以运行，但还是有点不对劲。如果重启应用程序（通过`cf restart`命令），重新登录，你会发现阅读列表清空了。你在重启前添加的书都不见了。

应用程序重启后数据消失，原因在于内嵌的H2数据库还在使用。我们可以通过Actuator的/health端点验证推测。它返回的信息大约是这样的：
```
{
  "status": "UP",
  "diskSpace": {
    "status": "UP",
    "free": 834236510208,
    "threshold": 10485760
  }, "db": {
    "status": "UP",
    "database": "H2",
    "hello": 1
  }
}
```
请注意`db.database`属性的值。它证实了我们之前的怀疑——果然用的是内嵌的H2数据库。我们需要修复这个问题。

实际上，Cloud Foundry以市集服务（marketplace services）的形式提供了一些数据库以供选择，包括MySQL和PostgreSQL。因为我们已经在项目里放了PostgreSQL的JDBC驱动，所以就使用市集里的PostgreSQL服务，名字是elephantsql。

elephantsql服务也有不少计划可选，小到开发用的小型数据库，大到工业级生产数据库。elephantsql的完整计划列表可以通过`cf marketplace`命令获得。
```
$ cf marketplace -s elephantsql
Getting service plan information for service elephantsql as craig@habuma.com... OK

service plan		   description			       free or paid
turtle		             Tiny Turtle	                 free
panda	               Pretty Panda	                   paid
hippo	                Happy Hippo	                   paid
elephant              Enormous Elephant                    paid
```
如你所见，比较严谨的生产级数据库计划都是要付费的。你可以选择你所期望的计划。我先假设你会选择免费的turtle。

创建数据库服务的实例，需要使用`cf create-service`命令，指定服务名、计划名和实例名。
```
$ cf create-service elephantsql turtle readinglistdb
Creating service readinglistdb in org habuma /
      space development as craig@habuma.com...
OK
```
服务创建后，需要通过`cf bind-service`命令将它绑定到我们的应用程序上。
```
$ cf bind-service sbia-readinglist readinglistdb
```  
将一个服务绑定到应用程序上不过就是为应用程序提供了连接服务的细节，这里用的是名为`VCAP_SERVICES`的环境变量。它不会通过修改应用程序来使用服务。

我们可以改写阅读列表应用程序，读取`VCAP_SERVICES`，使用其中提供的信息来连接数据库服务。但其实完全不用这么做。实际上，我们只需用`cf restage`命令重启应用程序就可以了：
```
$ cf restage sbia-readinglist
```
`cf restage`命令会让Cloud Foundry重新部署应用程序，并重新计算`VCAP_SERVICES`的值。如此一来，我们的应用程序会在Spring应用程序上下文里声明一个引用了绑定数据库服务的`DataSource` Bean，用它来替换原来的`DataSource` Bean。这样我们就能抛开内嵌的H2数据库，使用elephantsql提供的PostgreSQL服务了。

现在来试一下。登录应用程序，添加几本书，然后重启。重启之后你所添加的书应该还在列表里，因为它们已经被持久化在绑定的数据库服务里，而非内嵌的H2数据库里。再访问一下Actuator的/health端点，返回的内容能证明我们在使用PostgreSQL：
```
{
  "status": "UP",
  "diskSpace": {
    "status": "UP",
    "free": 834331525120,
    "threshold": 10485760
  }, "db": {
    "status": "UP",
    "database": "PostgreSQL",
    "hello": 1
  }
}
```
Cloud Foundry对Spring Boot应用程序部署而言是极佳的PaaS，Cloud Foundry与Spring项目搭配可谓如虎添翼。但Cloud Foundry并非Spring Boot应用程序在PaaS方面的唯一选择。让我们来看看如何将阅读列表应用程序部署到另一个流行的Paas平台：Heroku。

### 8.3.2 部署到Heroku

Heroku在应用程序部署上有一套独特的方法，不用部署完整的部署产物。Heroku为你的应用程序安排了一个Git仓库。每当你向仓库里提交代码时，它都会自动为你构建并部署应用程序。

如果还是解决不了问题，则需要先将项目目录初始化为Git仓库。
```
$ git init
```
这样Heroku的命令行工具就能自动把远程Heroku Git仓库添加到项目里。

现在可以通过Heroku的命令行工具在Heroku中设置应用程序了。这里使用`apps:create`命令。
```
$ heroku apps:create sbia-readinglist
```
这里我要求Heroku将应用程序命名为sbia-readinglist。这将成为Git仓库的名字，同时也是应用程序在herokuapps.com的子域名。你需要确定这个名字唯一，因为不能有同名应用程序。此外，你也可以让Heroku来替你生成一个独特的名字（比如fierce-river-8120或serene-anchorage-6223）。

`apps:create`命令会在[https://git.heroku.com/sbia-readinglist.git](https://git.heroku.com/sbia-readinglist.git)创建一个远程Git仓库，并在本地项目的Git配置里添加一个名为heroku的远程仓库引用。有了它就能通过`git`命令将我们的项目推送到Heroku了。

Heroku里的项目已经设置完毕，但我们现在还不能进行推送。Heroku需要你提供一个名为Procfile的文件，告诉Heroku应用程序构建后该如何运行。对于阅读列表应用程序而言，我们需要告诉Heroku，构建生成的WAR文件要当作可执行JAR文件来运行，这里使用`java`命令。***{![当前使用的项目会实际生成一个可执行的WAR文件。但对Heroku来说，它和可执行的JAR文件没什么区别。]}***假设应用程序是用Gradle来构建的，只需要如下一行内容的Procfile：
```
web: java -Dserver.port=$PORT -jar build/libs/readinglist.war
```
另一方面，如果你使用Maven来构建项目，JAR文件的路径就会有所不同。Heroku需要到target目录，而不是build/libs目录里寻找可执行WAR文件。具体如下：
```
web: java -Dserver.port=$PORT -jar target/readinglist.war
```
不管何种情况，你都需要像例子中那样设置`server.port`属性。这样内嵌的Tomcat服务器才能在Heroku分配的端口上（通过`$PORT`变量指定）启动。

我们差不多可以把应用程序推上Heroku了，但Gradle构建说明还要稍作调整。Heroku构建应用程序时，会执行一个名为`stage`的任务，因此需要在build.gradle里添加这个`stage`任务。
```
task stage(dependsOn: ['build']) {
}
```
如你所见，这个`stage`任务什么也没做，但依赖了`build`任务。于是，在Heroku使用`stage`任务构建应用程序会触发`build`任务，生成的JAR文件会放在build/libs目录里。

你还需要告诉Heroku用什么Java版本来构建并运行应用程序。这样Heroku才能用合适的版本来运行它。最简单的方法是在项目根目录里创一个名为system.properties的文件，在其中设置`java.runtime.version`属性：
```
java.runtime.version=1.7
```
现在就可以将项目推上Heroku了。和前面说一样，只需将代码推到远程Git仓库，Heroku会帮我们搞定其他事情。
```
$ git commit -am "Initial commit"
$ git push heroku master
```  
然后，Heroku会根据找到的构建说明文件，使用Maven或Gradle进行构建，再用`Procfile`里的指令来运行应用程序。就绪后，你可以用浏览器打开[http://{app name}.herokuapp.com](http://{app name}.herokuapp.com)。这里的{app name}就是你在`apps:create`里给应用程序起的名字。例如，我在部署时将应用程序命名为sbia-readinglist，所以它的URL就是[http://sbia-readinglist.herokuapps.com](http://sbia-readinglist.herokuapps.com)。

你可以在应用程序里随便点点，但要访问一下/health端点。`db.database`属性会告诉你应用程序正在使用内嵌的H2数据库。我们应该把它换成PostgreSQL服务。

我们可以通过Heroku命令行工具的`addons:add`命令创建并绑定一个PostgreSQL服务。
```
$ heroku addons:add heroku-postgresql:hobby-dev
```
这里我们要使用名为`heroku-postgresql`的附加服务。这是Heroku提供的PostgreSQL服务。我们还要求使用该服务的`hobby-dev`计划，这是免费的。

在PostgreSQL服务创建并绑定到应用程序后，Heroku会自动重启应用程序以保证绑定生效。但即便如此，我们在访问/health端点时仍然会看到应用程序还在使用内嵌的H2数据库。那是因为H2的自动配置仍然有效，谁也没告诉Spring Boot要用PostgreSQL代替H2。

一个办法是设置`spring.datasource.*`属性，做法和我们将应用程序部署到应用服务器上时一样。我们所需要的信息能在数据库服务的仪表板上找到，可以用`addons:open`命令打开仪表板。
```
$ heroku addons:open waking-carefully-3728
```  
在这个例子里，数据库实例的名字是waking-carefully-3728。该命令会在Web浏览器里打开仪表板页面，其中包含了你所需要的全部连接信息，包括主机名、数据库名和账户信息。总之，设置`spring.datasource.*`属性所需的一切信息都在这里了。

还有一个更简单的办法，与其自己查找那些信息，再设置到属性里，为什么不让Spring替我们查找信息呢？实际上，这就是Spring Cloud Connectors的用途。它可以用在Cloud Foundry和Heroku上，查找绑定到应用程序上的所有服务，并自动配置应用程序，以便使用那些服务。

我们只需在项目中加入Spring Cloud Connectors依赖即可。在Gradle项目里，在build.gradle中添加如下内容：
```
compile( "org.springframework.boot:spring-boot-starter-cloud-connectors")
```
如果你用的是Maven，则添加如下Spring Cloud Connectors`<dependency>`：
```
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-cloud-connectors</artifactId>
</dependency>
```
只有激活`cloud` Profile，Spring Cloud Connectors才会工作。要在Heroku里激活`cloud` Profile，可以使用`config:set`命令：
```
$ heroku config:set SPRING_PROFILES_ACTIVE="cloud"
```
现在项目里有了Spring Cloud Connectors依赖，`cloud` Profile也激活了。我们可以再推一次应用程序。
```
$ git commit -am "Add cloud connector"
$ git push heroku master
```
应用程序启动后，登入应用程序，查看/health端点。它应该显示应用程序已经连接到了PostgreSQL数据库：
```
"db": {
  "status": "UP",
  "database": "PostgreSQL",
  "hello": 1
}
```
现在我们的应用程序已经部署到云上，可以接受世界各地的请求了！

## 8.4 小结

Spring Boot应用程序的部署有多种方式，包括使用传统的应用服务器和云上的PaaS平台。在本章，我们了解了其中的一些部署方式，把阅读列表应用程序以WAR文件的方式部署到Tomcat和云上（Cloud Foundry和Heroku）。

Spring Boot应用程序的构建说明经常会配置为生成可执行的JAR文件。我们也看到了如何对构建进行微调，如何编写一个`SpringBootServletInitializer`实现，生成WAR文件，以便部署到应用服务器上。

随后，我们进一步了解了如何将应用程序部署到Cloud Foundry上。Cloud Foundry非常灵活，能够接受各种形式的Spring Boot应用程序，包括可执行JAR文件、传统WAR文件，甚至还包括原始的Spring Boot CLI Groovy脚本。我们还了解了Cloud Foundry如何自动将内嵌式数据源替换为绑定到应用程序上的数据库服务。

虽然Heroku不能像Cloud Foundry那样自动替换数据源的Bean，但在本章最后，我们还是看到了如何通过添加Spring Cloud Foundry库来实现一样的效果。这里使用绑定的数据库服务，而非内嵌式数据库。

在本章，我们还了解了如何在Spring Boot里使用Flyway和Liquibase这样的数据库迁移工具。在初次部署应用程序时，我们通过数据库迁移的方式完成了数据库的初始化，在后续的部署过程中，我们可以按需修改数据库。
