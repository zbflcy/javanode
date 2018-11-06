[TOC]
## Srping Boot 官方文档阅读

### Part II. Getting start

#### 11.3 Writting the code

##### @RestController and @RequestMapping
@RestController是构造性（stereotype）注解，提醒人们这个类扮演的角色是web controller,用来处理web 请求。@RestController 表示spring会直接返回字符串给请求者。  
@RequestMapping注解提供了路由信息。   
这两个都是Spring MVC的注解，并非Spirng Boot 特有。
##### @EnableAutoConfiguration
@EnableAutoConfiguration注解，表示，springboot会根据你自行添加的jar依赖配置spring。
#### 11.5创建可执行jar包  
* mvn package  
* java -jar

### Part III. Using Spring Boot
#### 13 Build Systems
##### 13.1 spring-boot-starter-parent 提供特性
  * 默认编译级别 java1.8
  * UTF-8 编码
  * 一个依赖管理节点，继承自spring-boot-dependencies pom，管理普通依赖的版本，允许你忽略<version>标签。
  * An execution of the repackage goal with a repackage execution id.用来生成可执行jar文件
  * 合理的资源过滤
  * 合理的插件管理
  * 针对application.properties和application.yml的筛选，包括特定profile的文件。  
>**Note**  
>注意需要在spring-boot-starter-parent指定唯一的version，这样，再引入其他starter的时候，可以忽略version numer

```xml
<!-- Inherit defaults from Spring Boot -->
<parent>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-parent</artifactId>
<version>2.1.0.RELEASE</version>
</parent>
```
可以通过重写property在自己的项目中覆盖特定依赖,比如将spring-data升级到另外一个版本，在pom.xml中如下：

```xml
<properties>
<spring-data-releasetrain.version>Fowler-SR2</spring-data-releasetrain.version>
</properties>
```

spring-boot-starter-parent采用了相当保守的java策略，使用最新的java版本，可以添加一个java.version属性
```xml
<properties>
    <java.version>1.8</java.version>
</properties>
```



##### 13.2在不使用parent POM的情况下玩转Spring Boot
并不是所有的人都喜欢集成spring-boot-starter-parent pom，这种情况下，可以使用一个 scope=import 的依赖来进行获取 dependency management的支持。
```xml
<dependencyManagement>
<dependencies>
<dependency>
<!-- Import dependency management from Spring Boot -->
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-dependencies</artifactId>
<version>2.1.0.RELEASE</version>
<type>pom</type>
<scope>import</scope>
</dependency>
</dependencies>
</dependencyManagement>
```
这种方式不支持重写属性的方式覆盖特性的依赖，为了达到同样的效果，可以在`dependencyManangement`中增加一个entry，添加的位置是在`spring-boot-dependencies`之前。同样，我们以spring-data升级为例。
```xml
<dependencyManagement>
Spring Boot Reference Guide
2.1.0.RELEASE Spring Boot 26
<dependencies>
<!-- Override Spring Data release train provided by Spring Boot -->
<dependency>
<groupId>org.springframework.data</groupId>
<artifactId>spring-data-releasetrain</artifactId>
<version>Fowler-SR2</version>
<type>pom</type>
<scope>import</scope>
</dependency>
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-dependencies</artifactId>
<version>2.1.0.RELEASE</version>
<type>pom</type>
<scope>import</scope>
</dependency>
</dependencies>
</dependencyManagement>
```  

##### 13.5 Starters启动器
Starters是一个依赖描述符的集合，你可以将它包含进项目中，这样添加依赖就非常方便。你可以获取所有Spring及相关技术的一站式服务，而不需要翻阅示例代码，拷贝粘贴大量的依赖描述符。例如，如果你想使用Spring和JPA进行数据库访问，只需要在项目中包含spring-boot-starter-data-jpa依赖，然后你就可以开始了。
>starters名字  
>所有的官方starters遵循以下命名格式：spring-boot-starter-\*，  
>自定义的启动器，不应该以spring-boot-starter开头，应该符以项目名称开头，比如 pro-spring-boot-starter。



* [Spring Boot Starters](https://www.nosuchfield.com/2017/10/15/Spring-Boot-Starters/)  
* [Spring Boot Starters启动器详解](https://blog.csdn.net/chszs/article/details/50610474)

#### 14 Structuring code
##### 14.2 Main class 的位置  
建议将Main class 放在root package下，main class通常包含 @SpringBootApplication 注解，并且，main class隐式的定义了基础的包搜索路径。例如，你正在开发一个JPA项目， spring 将搜索 @SpringBootApplication  注解所在的包，查找使用 @Entity 注解的类。将main class放到root package可以使 component scan (组建扫描)作用到整个项目中。
> **TIP**
>如果不想使用 @SpringBootApplication ,可以使用 @EnableAutoConfiguration 和 @ComponentScan 。  
>采用root package方式，你就可以使用 @ComponentScan 注解而不需要指定basePackage属性，也可以使用 @SpringBootApplication 注解，只要将main类放到root package中。

#### 15 配置类
Sping Boot 建议基于java的配置，通常将 @Configuration 放在定义了main方法的类中。
##### 15.1 引入额外的配置类
没有必要将所有的配置都放在同一个配置类中，使用 @Import 注解引入额外的配置类。也可以使用　@ComponentScan 自动查找spring组件，这其中就包含配置类（使用了 @Configuration ）
##### 15.2 引入xml配置文件
即使必须使用xml配置，spring boot也建议仍旧从一个 @Configuration 类开始，使用 @ImortResource 加载xml配置文件。

#### 16 Auto-configuration 自动配置
spring boot尝试根据你添加的jar依赖自动配置spring项目。  
实现自动配置有两种方式，在一个配置类（ @Configuration ）上使用 @EnableAutoConfiguration 或者 @SpringBootApplication 注解。只能使用其中任意一个，通常建议在主配置类上。

##### 16.1 替代自动配置
自动配置是非侵入性（non-invasive）,随时可以定义自己的配置类覆盖自动配置的特定部分。  
如果需要查看当前应用启动了哪些自动配置项，你可以在运行应用时打开--debug开关，这将为核心日志开启debug日志级别，并将自动配置相关的日志输出到控制台。  
##### 16.2 禁用特定的自动配置类
可以通过 @EnableAutoConfiguration  的 exclude 属性禁用特定的自动配置类。可以在exclude中配置多个需要禁用的配置类。
```java
@Configuration
@EnableAutoConfiguration(exclude={DataSourceAutoConfiguration.class})
public class MyConfiguration {
}
```
 如果该类不再classpath下面，还可以通过excludeName属性制定需要禁用配置类的全限定名。  
#### 17 Sping Beans and Dependency Injection
spring boot 允许使用任何标准的Spring框架技术去定义beans和它们注入的依赖。简单起见，我们经常使用@ComponentScan注解搜索beans，并结合@Autowired构造器注入。  
如果遵循以上的建议组织代码结构（将应用的main类放到包的最上层，即root package），那么你就可以添加@ComponentScan注解而不需要任何参数，所有应用组件（@Component, @Service, @Repository, @Controller等）都会自动注册成Spring Beans。
>**注意使用构建器注入允许字段被标记为final，这意味着后续是不能改变的。**

#### 18 使用 @SpringBootApplication 注解
@SpringBootApplication 等价于使用　@Configuration 、 @EnableAutoConfiguration 和 @ComponentScan 三个注解的默认属性。  
* @Configuration ：允许注册额外的bean和导入附加的配置
* @EnableAutoConfiguration ：启动sping boot 自动配置机制
* @ComponentScan ： 允许扫描所在package

#### 20 developer tools开发者工具
spring-boot-devtools 模块提供了development-features，添加以下依赖获得devtools
```xml
<dependencies>
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-devtools</artifactId>
<optional>true</optional>
</dependency>
</dependencies>
```
>**Node**  
>在运行完整的打包过的程序时，开发者工具会自动禁用。如果程序是通过java -jar启动或者通过特殊的类加载器启动，会被认为是产品应用，禁用开发者工具。  
>为了防止devtools传递到项目中的其他模块，设置optional为true  

>**Tip**  
>Repackaged archives 默认不包含devtools。如果想使用开发者工具，需要禁用内置属性excludeDevtools  

##### 20.1 默认属性
1. 尽管缓存在生产环境中很有用，但是在开发中可能会产生反作用，阻止你看到最新的改变，所以，sping boot devtools默认禁用缓存。可以在application.xml中配置缓存选项。
2. spring boot developer tools 默认将web logging group的日志设置为DEBUG，可以通过sping.http.log-request-details配置该属性。
3. 如果不想使用默认属性，可以在application.properties将spring.devtools.add-properties设置成false

##### 20.2 自动重启
如果程序使用spring-boot-devtools，当classpath下面的文件发生改变时会自动重启。  
默认情况下，classpath下所有的文件夹都会被监听。但是特定的资源，比如静态资源是不需要重启程序的。  
>**IDEA Trgger a restart**  
>1. application.properties中配置spring.devtools.restart.enabled=true  
>2. File-Settring-Complier-Build Project automatically
>3. ctrl+shift+alt+/,勾选Complier autoMake allow when app running  

>**Node**  
>* 可以通过禁用shutdown hook关闭自动重启，SpringApplication.setRegisterShutdownHook(false)  
>* devtools自动重启忽略以spring-boot, spring-boot-devtools, spring-
boot-autoconfigure, spring-boot-actuator, and spring-boot-starter开头的项目

>**restart and reload**  
>spring boot 通过使用两个classloaders实现restart，base classloader负责加载不变的类，比如第三方jar包。restart classloader负责加载当前开发的类。当项目重启时，旧的restart classloader会被丢弃，然后新建一个，这种方法保证应用程序重启通常比冷启动快很多，因为base classloader已经可用。  
>如果重启很慢或者遇到加载问题，可以考虑使用reload技术，比如JRebel，通过在加载类时重写它们，使它们更易于重新加载

**Logging changes in condition evaluaiton**   
每次重启，默认都会记录显示condition evaluation delta 的报告。通过这个报告，可以看到程序  
在application.properties中配置`spring.devtools.restart.log-condition-evaluation-delta=false`禁用该报告。  

**Excluding Resources**  
默认情况下，/META-INF/maven,
/META-INF/resources, /resources, /static, /public, or /templates文件夹下面的更改并不会触发restart，但是会触发reload。可以使用spring.devtools.restart.exclude属性配置需要排除的文件，比如想要移除对/static和/pulic的监听,`spring.devtools.restart.exclude=static/**,pulic/**` 。  
如果想保持默认，增加额外排除项，使用`sping.devtools.restart.additional-exclude`。

**Watch Additional Paths**
如果想监听不在classpath下面的文件或者文件夹，发生改变的时候，触发程序restart或者reload，可以在application.properties中配置spring.devtools.restart.additional-paths路径

**禁止重启**    
在application.propertis中配置spring.devtools.restart.enabled=false。仅仅这样配置的话，restart classloader仍然会初始化，但是不会匹配任何修改。  
想完全禁止restart，在主程序调用SpringApplication.run(...)之前，将System的spring.devtools.restart.enabled属性设置为false。
```java
public static void main(String[] args) {
  System.setProperty("spring.devtools.restart.enabled", "false");
  SpringApplication.run(MyApp.class, args);
}
```

**使用Trigger File**  
使用IDE开发的时候，相比频繁的触发程序restart，更想只在特定的时间触发，可以使用"Tigger file"。要想触发restart，必须修改trigger file。更改文件只会触发检查，并且只有在Devtools检测到有必要的时候才会重新启动。  
要想使用trgger file 在application.properties中，将spring.devtools.restart.trigger-file 指定为trigger file的路径。  
>**TIP**  
>可以将pring.devtools.restart.trigger-file设置为全局，以便所有项目都以相同的方式运行。

**自定义类加载器**

如上所述,spring boot通过使用两个类加载器实现自动重启功能，对于大多数程序来说，这种方式很适用，但是有时可能造成加载问题。默认情况下，所有在IDE打开的程序都会通过restart classloader加载，所有常规的.jar文件使用base classloader加载。如果你工作在一个多模块的项目下，并且不是每个模块都导入IDE里，你可能需要自定义一些东西。  
可以创建META-INF/spring-devtools.properties文件，里面包含以restart.exclude和restart.include为前缀的属性。include元素指的是需要通过restart classloader加载的类，exclude元素指的是通过base classloader加载的类。属性值可以是正则表达式。
```java
restart.exclude.companycommonlibs=/mycorp-common-[\\w-]+\.jar
restart.include.projectcommon=/mycorp-myproj-[\\w-]+\.jar
```

**Node**  
所有属性的keys必须唯一，只要以restart.include.或restart.exclude.开头都会考虑进去。所有来自classpath的META-INF/spring-devtools.properties都会被加载，你可以将文件打包进工程或工程使用的库里。  
##### 20.3 LiveReload
spirng-boot-devtools内嵌了一个LiveReload server，当资源更新的时候，用来触发浏览器刷新。  
如果想禁用LiveReload，设置`spring.devtools.livereload`为false。  
>**Node**
>只能同时运行一个livereload server，在启动项目之前，确保没有其他的livereload server正在运行。
##### 20.4 全局设置
在$home目录下建立一个.spring-boot-
devtools.properties，该文件中的属性会应用到本机中的所有使用devtoolos的SpringBoot项目。比如，你想配置trigger file.  
~/.spring-boot-devtools.properties
```java
spring.devtools.reload.trgger-file=.reloadtrigger
```
##### 20.5远程调用（没看懂啦）
Spring Boot开发者工具并不仅限于本地开发，在运行远程应用时你也可以使用一些特性。远程支持是可选的。远程调用是可选的，如果想使用这个功能，要确保在repackaged archive（比如jar，war）中包含devtools.
```xml
<build>
  <plugins>
    <plugin>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-maven-plugin</artifactId>
      <configuration>
        <excludeDevtools>false</excludeDevtools>
      </configuration>
    </plugin>
  </plugins>
  </build>
```
然后设置`sping.devtools.remote.secret`属性。  
```
spring.devtools.remote.secret=mysecret
```
>**Warning**
> 在远程应用上启用spring-boot-devtools有一定的安全风险，生产环境中最好不要使用  

远程开发者工具支持分为两个部分：一个是接受连接的服务端端点，一个事运行在本地IDE中的客户端程序。设置`spring.devtools.remote.secret`后，服务端会自动启用，客户端组件则需要手动启动。

**运行远程客户端应用**
远程客户端应用程序（remote client application）需要在IDE中运行，你需要使用跟将要连接的远程应用相同的classpath运行org.springframework.boot.devtools.RemoteSpringApplication，传参为你要连接的远程应用URL。例如，你正在使用Eclipse或STS，并有一个部署到Cloud Foundry的my-app工程，远程连接该应用需要做以下操作：
* 从Run菜单选择Run Configurations…。
* 创建一个新的Java Application启动配置（launch configuration）。
* 浏览my-app工程。
* 将org.springframework.boot.devtools.RemoteSpringApplication作为main类。
* 将https://myapp.cfapps.io作为参数传递给RemoteSpringApplication（或其他任何远程URL）。

>**NODE**  
因为远程客户端使用的classpath跟真实应用相同，所以它能直接读取应用配置，这就是spring.devtools.remote.secret如何被读取和传递给服务器做验证的。
强烈建议使用https://作为连接协议，这样传输通道是加密的，密码也不会被截获。
如果需要使用代理连接远程应用，你需要配置spring.devtools.remote.proxy.host和spring.devtools.remote.proxy.port属性。

**远程更新**  
远程客户端将监听应用的classpath变化，任何更新的资源都会发布到远程应用，并触发重启，这在你使用云服务迭代某个特性时非常有用。通常远程更新和重启比完整rebuild和deploy快多了。
>**NODE**  
文件只有在远程客户端运行时才监控。如果你在启动远程客户端之前改变一个文件，它是不会被发布到远程server的。
