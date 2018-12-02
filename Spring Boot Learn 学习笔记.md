## Spring Boot Learn 学习笔记

### 单元测试
spring boot 取消了 @SpringApplicationConfiguration 这个注解，使用@SpringBootTest 就可以了

### application.properties
#### 使用随机数
```
# 随机字符串
com.didispace.blog.value=${random.value}
# 随机int
com.didispace.blog.number=${random.int}
# 随机long
com.didispace.blog.bignumber=${random.long}
# 10以内的随机数
com.didispace.blog.test1=${random.int(10)}
# 10-20的随机数
com.didispace.blog.test2=${random.int[10,20]}
```
#### 参数相互引用
```
com.didispace.blog.name=程序猿DD
com.didispace.blog.title=Spring Boot教程
com.didispace.blog.desc=${com.didispace.blog.name}正在努力写《${com.didispace.blog.title}》
```
#### 命令行设置属性
在命令行运行时，连续的两个减号--就是对application.properties中的属性值进行赋值的标识。所以，java -jar xxx.jar --server.port=8888命令，等价于我们在application.properties中添加属性server.port=8888，该设置在样例工程中可见，读者可通过删除该值或使用命令行来设置该值来验证。  
通过命令行来修改属性值固然提供了不错的便利性，但是通过命令行就能更改应用运行的参数，那岂不是很不安全？是的，所以Spring Boot也贴心的提供了屏蔽命令行访问属性的设置，只需要这句设置就能屏蔽：SpringApplication.setAddCommandLineProperties(false)。
#### 多环境配置
在Spring Boot中多环境配置文件名需要满足application-{profile}.properties的格式，其中{profile}对应你的环境标识，比如：  
* application-dev.properties：开发环境
* application-test.properties：测试环境
* application-prod.properties：生产环境
  
至于哪个具体的配置文件会被加载，需要在application.properties文件中通过`spring.profiles.active`属性来设置，其值对应{profile}值。  
#### 全小写配置和移除特殊字符
application.properties配置均以全小写的方式匹配，并且移除了特殊字符。下面4种配置
方式是等价的。
```
spring.jpa.databaseplatform=mysql
spring.jpa.database-platform=mysql
spring.jpa.databasePlatform=mysql
spring.JPA.database_platform=mysql
```
#### List类型
properties文件中使用[]定位列表类型，比如：  
```
spring.my-example.url[0]=http://example.com
spring.my-example.url[1]=http://spring.io
```
也可以使用逗号分配的配置方式，上面的配置方式同下面是等价的。  
```
spring.my-example.url=http://example.com,http://spring.io
```
在spring-boot 2.0中，对列表的配置必须是连续的，不然会抛出UnboundConfigurationPropertiesException异常，所以下面的配置是不允许的。  
```
foo[0]=a
foo[2]=b
```
