[TOC]


# Part IV. Spring Boot Features

## 26. Logging
spring boot 内部日志 使用 Commons Logging，但是开放底层日志实现。默认的配置提供了java Util Logging,Log4J2和Logback.各种情况，logger预设使用控制台输出，也可以使用文件输出。  

如果使用“Starters”，默认使用logback。同时，包含合适的Logback routing确保使用其他dependent libraries（Java util logging, commons loggings,log4j 或者slf4j）正确工作。  

### 26.1 日志格式
spring boot 默认的日志输出类似下面的例子：


```
2014-03-05 10:57:51.112  INFO 45469 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet Engine: Apache Tomcat/7.0.52
2014-03-05 10:57:51.253  INFO 45469 --- [ost-startStop-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2014-03-05 10:57:51.253  INFO 45469 --- [ost-startStop-1] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 1358 ms
2014-03-05 10:57:51.698  INFO 45469 --- [ost-startStop-1] o.s.b.c.e.ServletRegistrationBean        : Mapping servlet: 'dispatcherServlet' to [/]
2014-03-05 10:57:51.702  INFO 45469 --- [ost-startStop-1] o.s.b.c.embedded.FilterRegistrationBean  : Mapping filter: 'hiddenHttpMethodFilter' to: [/*]
```
下面的item将会被输出：

* Date and time :精确到毫秒，且易于排序
* log level : ERROR,WARN,INFO,DEBUG或者TRACE
* Process ID
* --- 分隔符，用于区分实际日志信息开头
* 线程名：包括在方括号中，（控制台输出可能会被截断）
* logger名： 通常是源class的类名。
* 日志信息

>**Note**  
>Logback 没有FATAL级别，会映射到ERROR。


### 26.2 控制台输出
默认的日志配置在写日志消息时会将他们回显到控制台。级别为ERROR, WARN和INFO的消息会被记录。可以使用`--debug`开启程序激活“debug” 别日志记录
```
$ java -jar myapp.jar --debug
```

>**Note** 
>也可以在application.properties中指定debug=true

当开启debug模式的时候，一系列核心logger（内嵌的容器，Hiernate和spring boot）会记录更多的日志，但是并不输出所有的信息。  

相应地，你可以在启动应用时，通过--trace（或在application.properties设置trace=true）启用"trace"模式，该模式能够追踪核心loggers（内嵌容器，Hibernate生成的schema，Spring全部的portfolio）的所有日志信息。

#### color-coded output
如果你的终端支持ANSI，彩色打印会有助于阅读。可以通过设置`spring.output.ansi.enabled`为true覆盖默认设置。  

彩色编码（Color coding）使用%clr表达式进行配置，在其最简单的形式中，转换器会根据日志级别使用不同的颜色输出日志，例如：
```
    %clr(%5p)
```

日志级别到颜色的映射如下：  

| Level | Color |
| - | - |
| FATAL | RED |
| ERROR | RED |
| WARN | YELLOW |
| INFO | GREEN |
| DEBUG | GREEN |
| TRACE | GREEN |

另外，在转换时你可以设定日志展示的颜色或样式，例如，让文本显示成黄色：
```
%clr(%d{yyyy-MM-dd HH:mm:ss.SSS}){yellow}
```

支持的颜色如下:

* blue
* cyan
* faint
* green
* magenta
* red
* yellow

### 26.3 文件输出
默认情况下，Spring boot日志只输出到控制台，而不写日志文件。如果想额外写日志i文件，需要设置`logging.file或者logging.path`属性。  

下面的表格显示了 `logging.*`属性如何一起使用

| logging.file | logging.path | example | Description |
| - | - | - | - |
| (none) | (none) |   | 只记录在控制台 |
| Specific file | (none) | my.log |  写到特定的日志文件，名称可以是精确的位置或相对于当前目录 |
| (none) | Specific direcitory | /var/log | 写到特定的日志文件，名称可以是精确的位置或相对于当前目录 |


日志文件每达到10M就会被分割，跟控制台一样，默认记录ERROR, WARN和INFO级别的信息。 Size limits 可以通过使用`logging.file.max-size`属性修改。之前的存档能永久保存，除非设置了`logging.file.max-history`

>**Note** 
> 日志框架在应用程序的早期就被初始化，因此 不能使用通过`\@PropertySource`加载的属性文件中的logging properties。  


### 26.4 日志级别

所有Spring Boot支持的日志系统都可以在Spring Environment中设置级别（application.properties里也一样），设置格式为`logging.level.<logger-name>=LEVEL`，其中LEVEL是TRACE, DEBUG, INFO, WARN, ERROR, FATAL, OFF之一,root logger 可是使用 `logging.level.root`配置。

以下是application.properties示例：

```properties
logging.level.root=WARN
logging.level.org.springframework.web=DEBUG
logging.level.org.hibernate=ERRO
```

### 26.5 日志分组
将日志分组通常是很有用的，这样可以一次性配置他们。比如，你一般会为所有tomcat相关logger修改日志级别，但是记不住顶级包（top level packages）.  

spring boot 允许你在你的spring environment 定义你的日志分组。比如，你在application.properties中定义一个"tomcat"分组。  
```properties
    logging.group.tomcat=org.apache.catalina, org.apache.coyote, org.apache.tomcat
```
定义之后，你可以只用一行就为分组内所有的loggers修改级别。  
```properties
    logging.level.tomcat=TRACE
```

Spring boot 预定义下面的日志分组，你可以直接使用：  

| **NAME** | **Loggers** |
| - | - |
| web | org.springframework.core.codec, org.springframework.http,
org.springframework.web | 
| sql | org.springframework.jdbc.core, org.hibernate.SQL |

### 26.6 自定义日志配置
通过将相应的库添加到classpath下面可以激活各种日志系统，然后在classpath根目录下面提供合适的日志配置文件可以进一步自定义日志系统，也可以spring environment 属性`logging.confg`指定配置文件。  

可以通过指定`org.springframework.boot.logging.LoggingSystem`强制spring boot 使用特定的日志系统。该属性值应该是一个日志系统实现的全限定名。如果是none，则禁用springboot的日志配置。  

>**Note** 
>由于日志初始化早于ApplicationContext的创建，所以不可能通过\@PropertySources指定的Spring \@Configuration文件控制日志。修改或完全禁用logging system 唯一的方式就是通过`System properties`。  

根据不同的日志系统，加载下面的配置文件

| logging System | Customization |
| - | - |
| logback | logback-spring.xml, logback-spring.groovy, logback.xml, or logback.groovy |
| log4j2 | log4j2-spring.xml or log4j2.xml |
| JDK(Java Util logging) | logging.properties |

> **Note**  
> 如果可能的话，建议使用带有`-spring`后缀配置文件（比如，logback-spring.xml 而不用logback.xml）。如果使用标准的配置文件，spring不能完全控制日志初始化。


>**Warning**  
>如果想在日志属性中使用占位符，你需要使用Spring Boot的语法，而不是底层框架的语法。尤其是使用Logback时，你需要使用:作为属性名和默认值的分隔符，而不是:-

### 26.7 日志扩展
Spring Boot包含很多有用的Logback扩展，你可以在logback-spring.xml配置文件中使用它们  

>**Note**  

>你不能在标准的logback.xml配置文件中使用扩展，因为它加载的太早了，不过可以使用logback-spring.xml，或指定logging.config属性。  

这些扩展不能同logback `configuration scanning`一起使用，不然会报和下面类似的错误：  
```
ERROR in ch.qos.logback.core.joran.spi.Interpreter@4:71 - no applicable action for [springProperty],
current ElementPath is [[configuration][springProperty]]
ERROR in ch.qos.logback.core.joran.spi.Interpreter@4:71 - no applicable action for [springProfile],
current ElementPath is [[configuration][springProfile]]
```

#### ProFile-specific Configuration
`<springProfile>`标签可用于根据激活的Spring profiles，选择性的包含或排除配置sections。Profile section可以放在`<configuration>`元素内的任何地方.使用name属性执行那些profiles接受这个配置。`<springProfile>`可以包含一个简单的profile name，或者是一个profile expression. profile express 允许更复杂的profile 逻辑，比如`production & (eu-central | eu-west)`

```xml
springProfile name="staging">
    <!-- configuration to be enabled when the "staging" profile is active -->
</springProfile>

<springProfile name="dev, staging">
    <!-- configuration to be enabled when the "dev" or "staging" profiles are active -->
</springProfile>

<springProfile name="!production">
    <!-- configuration to be enabled when the "production" profile is not active -->
</springProfile>
```

#### Envirionment Properties
`<springProperty>`标签允许你从Spring Environment读取属性，以便在Logback中使用.如果你想在logback配置获取application.properties中的属性值，该功能就很有用。该标签工作方式跟Logback标准`<property>`标签类似,不是直接指定value值，你需要定义属性的source(from the environment)。如果你需要在其他scope（不是local scope ）存储属性使用，可以使用scope attribute。如果你需要一个fallback value(没有在 environment设置)，可以使用default attribute。

```xml
<springProperty scope="context" name="fluentHost" source="myapp.fluentd.host"
defaultValue="localhost"/>
<appender name="FLUENT" class="ch.qos.logback.more.appenders.DataFluentAppender">
<remoteHost>${fluentHost}</remoteHost>
...
</appender>
```

>**Note（没懂）**  
> source必须以中划线的方式指定，（比如 my.property-name）。properties can be added to the Environment by using the relaxed rules