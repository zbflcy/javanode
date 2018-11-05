[toc]
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
spring-boot-devtools 模块提供了development-features
