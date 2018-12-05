[TOC]


# 28 Developing Web Application

Spring Boot非常适合开发web应用程序，你可以使用内嵌的Tomcat，Jetty或Undertow、netty轻轻松松地创建一个HTTP服务器。大多数的web应用都可以使用`spring-boot-starter-web`模块进行快速搭建和运行。你也可以使用`spring-boot-starter-webflux`创建一个reactive web应用。  

## 28.1 The “Spring Web MVC Framework”

Spring Web MVC框架(通常简称为 Spring MVC)是一个rich 'model view controller'web 框架，允许用户创建特定的\@Controller或者\@RestController beans 处理传入的http请求。通过@RequestMapping注解可以将控制器中的方法映射到相应的HTTP请求。  

下面是一个\@RestController（ serves JSON data）例子：
```java
@RestController
@RequestMapping(value="/users")
public class MyRestController {
	@RequestMapping(value="/{user}", method=RequestMethod.GET)
	public User getUser(@PathVariable Long user) {
		// ...
	}
	@RequestMapping(value="/{user}/customers", method=RequestMethod.GET)
	List<Customer> getUserCustomers(@PathVariable Long user) {
		// ...
	}
	@RequestMapping(value="/{user}", method=RequestMethod.DELETE)
	public User deleteUser(@PathVariable Long user) {
		// ...
	}
}
```

### Spring MVC 自动配置

Spring Boot 为Spring MVC提供了自动配置，这些配置适用于大部分应用。  

自动配置在Spring的默认配置之上添加了以下特性:

* 注册了ContentNegotiatingViewResolver和BeanNameViewResolver beans
* 支持静态资源访问，其中包括对WebJars的支持
* 主动注册了Converter、Converter和Converter beans
* 静态index.html 首页的支持。
* 支持自定义favicon.ico
* ConfigurableWebBindingInitializer 的自动使用

如果想保持Spring Boot MVC特性，增加额外的MVC配置（比如(interceptors, formatters, view controllers, and other features），可以通过添加自己的 WebMvcConfigurer 并使用\@Configuration注解，但是不要使\@EnableWebMvc  

如果想提供自定义的 RequestMappingHandlerMapping, RequestMappingHandlerAdapter, or
ExceptionHandlerExceptionResolver实例，可以声明一个WebMvcRegistrationsAdapter 实例。  

如果想完全控制Spring MVC，则在自己的WebMvcConfigurer上使用\@Configuration + \@EnableWebMVC注解 。

### HttpMessageConverters

Spring MVC 使用HttpMessageConverters 接口转换HTTP请求和响应。合适的默认配置可以开箱即用。例如对象自动转换为JSON（使用Jackson库）或XML（如果Jackson XML扩展可用，否则使用JAXB），字符串默认使用UTF-8编码。  

可以使用Spring Boot的`HttpMessageConverters`类添加或自定义 Converters：

```java
import org.springframework.boot.autoconfigure.web.HttpMessageConverters;
import org.springframework.context.annotation.*;
import org.springframework.http.converter.*;
@Configuration
public class MyConfiguration {
	@Bean
	public HttpMessageConverters customConverters() {
		HttpMessageConverter<?> additional = ...
		HttpMessageConverter<?> another = ...
		return new HttpMessageConverters(additional, another);
	}
}
```
上下文中出现的所有HttpMessageConverter bean都将添加到converters列表，你可以通过这种方式覆盖默认的转换器列表（converters）。

### Custom JSON Serializers and Deserializers

如果使用Jackson序列化，反序列化JSON数据，你可能想编写自己`JsonSerializer`或者`JsonDesesrializer`类。通常注册Jackson自定义Serializers 要通过 一个module，但是Spring Boot 提供了`\@JsonComponent`，可以更简单的将Serializers注册为Spring Beans.   

可以直接在`JsonSerializer`或者`JsonDeserializer`实现类上使用`\@JsonComponent`注解，也可以在包含 serializers/deserializers 的外部类上使用：  

```java
import java.io.*;
import com.fasterxml.jackson.core.*;
import com.fasterxml.jackson.databind.*;
import org.springframework.boot.jackson.*;
@JsonComponent
public class Example {
	public static class Serializer extends JsonSerializer<SomeObject> {
		// ...
	}
	public static class Deserializer extends JsonDeserializer<SomeObject> {
		// ...
	}
}
```

所有ApplicationContext 中的`\@JsonComponen` beans 都会被同jsackson一起自动注册，因为 `\@JsonComponen` 使用了@Component 注解，所以适用component-scanning 规则。  

### MessageCodesResolver

Spring MVC有一个实现策略，用于从绑定的errors产生用来渲染错误信息的错误码：`MessageCodesResolver`。Spring Boot会自动为你创建该实现，只要设置`spring.mvc.message-codes-resolver.format`属性为`PREFIX_ERROR_CODE`或`POSTFIX_ERROR_CODE`（具体查看`DefaultMessageCodesResolver.Format`枚举值）。

### Static Content

默认情况下，Spring Boot从classpath下的/static（/public，/resources或/META-INF/resources）文件夹，或从ServletContext根目录提供静态内容。这是通过Spring MVC的ResourceHttpRequestHandler实现的，你可以自定义WebMvcConfigurerAdapter并覆写addResourceHandlers方法来改变该行为（加载静态文件）  

在单机web应用中，容器会启动默认的servlet，并用它加载ServletContext根目录下的内容以响应那些Spring不处理的请求。大多数情况下这都不会发生（除非你修改默认的MVC配置），因为Spring总能够通过DispatcherServlet处理这些请求。  

静态资源默认被映射到`/**`，可以配置`sping.mvc.static-path-pattern`属性修改，比如，将所有的资源重新定位到`/resources/**`：`spring.mvc.static-path-pattern=/resources/**`  

可以使用`spring.resources.static-locations`（使用目录位置列表替换默认值）属性自定义静态资源文件的位置，SevletContext Path("/")则会自动添加。  

除了上述提到的"standard" 静态资源位置，有个例外情况是Webjars内容。任何在`/webjars/**`路径下的资源都将从jar文件中提供，只要它们以Webjars的格式打包。

>**Tip**  
>
>如果你的应用将被打包成jar，那就不要使用src/main/webapp文件夹。尽管该文件夹是通常的标准格式，但它仅在打包成war的情况下起作用，在打包成jar时，多数构建工具都会默认忽略它。  

Spring Boot也支持 Spring MVC提供的高级资源处理特性，可用于清除缓存的静态资源或对WebJar使用版本无感知的URLs。  

如果想使用针对WebJars版本无感知的URLs（version agnostic），只需要添加`webjars-locator`依赖，然后声明你的Webjar。以jQuery为例，`/webjars/jquery/dist/jquery.min.js`实际为`/webjars/jquery/x.y.z/dist/jquery.min.js`，x.y.z为Webjar的版本。

>**Note**  
>
>如果使用JBoss，你需要声明`webjars-locator-jboss-vfs`依赖而不是`webjars-locator`，否则所有的Webjars将解析为`404`  

以下的配置为所有的静态资源提供一种缓存清除（cache busting）方案，实际上是将内容hash添加到URLs中，比如`<link href="/css/spring-2a2d595e6ed9a0b24f027f2b63b134d6.css"/>`：  

```properties
spring.resources.chain.strategy.content.enabled=true
spring.resources.chain.strategy.content.paths=/**
```

>**Note** 
>
>实现该功能的是`ResourceUrlEncodingFilter`，它在模板运行期会重写资源链接，Thymeleafh和FreeMarker会自动配置该filter，JSP需要手动配置。其他模板引擎还没自动支持，不过你可以使用ResourceUrlProvider自定义模块宏或帮助类。  

当使用比如JavaScript模块加载器动态加载资源时，重命名文件是不行的，这也是提供其他策略并能结合使用的原因。下面是一个"fixed"策略，在URL中添加一个静态version字符串而不需要改变文件名：  
```properties
spring.resources.chain.strategy.content.enabled=true
spring.resources.chain.strategy.content.paths=/**
spring.resources.chain.strategy.fixed.enabled=true
spring.resources.chain.strategy.fixed.paths=/js/lib/
spring.resources.chain.strategy.fixed.version=v12
```

使用用以上策略，JavaScript模块加载器加载`/js/lib/`下的文件时会使用一个固定的版本策略`/v12/js/lib/mymodule.js`，其他资源仍旧使用内容hash的方式<`link href="/css/spring-2a2d595e6ed9a0b24f027f2b63b134d6.css"/>`。

### Welcome Page

Spring Boot 同时支持静态首页和模板首页。Spring Boot 首先在配置好的静态内容位置寻找`index.html`，如果没有，就寻找一个`index`模板页。  

同时有的话，会选择模板首页。  

### Custom Favicon

Spring Boot 在配置好的静态内容位置和classpath(就按照这顺序搜索)寻找`favicon.ico`作为项目的favicon。  

### Path Matching and Content Negotiation

Spring MVC 通过查询请求路径可以将传入的HTTP请求映射到handler中，然后将其匹配到应用中定义的mappings中，比如`\@GetMapping`注解的Controller方法。  

Spring Boot 默认选择禁用后缀模式匹配，意味着像`Get /projects/spring-boot.json`这样的请求并不会匹配到 `\@GetMapping("/projects/spring-boot")` mappings中，这个特性在过去HTTP客户端不发送“Accept”请求头的时候很有用。   

我们需要确保向客户端发送正确的`Content Type`，`Content
Negotiation`更值得信赖。

有其他的方式处理那写没并不发送“Accept”请求头的HTTP客户端。不采用`suffix matching`,而使用请求参数确保`GET /projects/spring-boot?format=json`这样的请求能映射到`\@Getmapping("/projects/spring-boot")`:
```properties
spring.mvc.contentnegotiation.favor-parameter=true

# We can change the parameter name, which is "format" by default:
# 我们可以改变参数名，默认是“format”
# spring.mvc.contentnegotiation.parameter-name=myparam

# We can also register additional file extensions/media types with:
# 同样可以注册额外的文件扩展名类型
spring.mvc.contentnegotiation.media-types.markdown=text/markdown
```

如果想继续使用后缀模式匹配，需要如下配置：  
```properties
spring.mvc.contentnegotiation.favor-path-extension=true
spring.mvc.pathmatch.use-suffix-pattern=true
```

有一种更为安全的方式，就是不开始所有的后缀模式，而只支持注册的后缀模式:  
```properties
spring.mvc.contentnegotiation.favor-path-extension=true
spring.mvc.pathmatch.use-registered-suffix-pattern=true

# You can also register additional file extensions/media types with:
# spring.mvc.contentnegotiation.media-types.adoc=text/asciidoc

```

### ConfigurableWebBindingInitializer

Spring MVC使用`WebBindingInitializer`为每个特殊的请求初始化相应的`WebDataBinder`，如果你创建自己的`ConfigurableWebBindingInitializer @Bean`，Spring Boot会自动配置Spring MVC使用它。

### Tempalte Engines

正如REST web服务，你也可以使用Spring MVC提供动态HTML内容。Spring MVC支持各种各样的模板技术，包括Thymeleaf,FreeMarker和JSPs，很多其他的模板引擎也提供它们自己的Spring MVC集成。  

Spring Boot为以下的模板引擎提供自动配置支持：

* FreeMarker
* Groovy
* Thymeleaf
* Mustache

>**Tip**  
> 
>由于在内嵌servlet容器中使用JSPs存在一些已知的限制，所以建议尽量不使用它们

使用以上引擎中的任何一种，并采用默认配置，则模块会从src/main/resources/templates自动加载。

>**Tip**
>
>IntelliJ IDEA根据你运行应用的方式会对classpath进行不同的排序。在IDE里通过main方法运行应用，跟从Maven，或Gradle，或打包好的jar中运行相比会导致不同的顺序，这可能导致Spring Boot不能从classpath下成功地找到模板。如果遇到这个问题，你可以在IDE里重新对classpath进行排序，将模块的类和资源放到第一位。或者，你可以配置模块的前缀为classpath*:/templates/，这样会查找classpath下的所有模板目录。  

### Error  Handling

Spring Boot 默认提供`/error` mapping以合适的方式处理所有的error，并将它注册为servlet容器中全局的 错误页面。对于机器客户端（相对于浏览器而言，浏览器偏重于人的行为），它会产生一个具有详细错误，HTTP状态，异常信息的JSON响应。对于浏览器客户端，它会产生一个白色标签样式（whitelabel）的错误视图，该视图将以HTML格式显示同样的数据（可以添加一个解析为'error'的View来自定义它）。  

为了完全替换默认的行为，你可以实现ErrorController，并注册一个该类型的bean定义，或简单地添加一个ErrorAttributes类型的bean以使用现存的机制，只是替换显示的内容。

>**Tip**
> 
>BasicErrorController可以作为自定义ErrorController的基类。如果你想添加对新Content type的处理（默认处理text/html），这会很有帮助。你只需要继承BasicErrorController，添加一个public方法，并注解带有produces属性的@RequestMapping，然后创建该新类型的bean。  

你也可以定义一个使用\@ControllerAdvice注解的类，进而自定义某个特殊controller或exception类型返回的JSON文本。  
```java
@ControllerAdvice(basePackageClasses = AcmeController.class)
public class AcmeControllerAdvice extends ResponseEntityExceptionHandler {
	@ExceptionHandler(YourException.class)
	@ResponseBody
	ResponseEntity<?> handleControllerException(HttpServletRequest request, Throwable ex) {
		HttpStatus status = getStatus(request);
		return new ResponseEntity<>(new CustomErrorType(status.value(), ex.getMessage()), status);
	}
	private HttpStatus getStatus(HttpServletRequest request) {
		Integer statusCode = (Integer) request.getAttribute("javax.servlet.error.status_code");
		if (statusCode == null) {
			return HttpStatus.INTERNAL_SERVER_ERROR;
		}
		return HttpStatus.valueOf(statusCode);
	}
}
```

在上面的例子中，如果跟`FooController`相同package的某个controller抛出YourException，一个`CustomerErrorType`类型的POJO的json展示将代替`ErrorAttributes`展示。  

#### Custom Error Pages

如果想为某个给定的状态码展示一个自定义的HTML错误页面，你需要将文件添加到/error文件夹下。错误页面既可以是静态HTML（比如，任何静态资源文件夹下添加的），也可以是使用模板构建的，文件名必须是明确的状态码或一系列标签。  

例如，映射404到一个静态HTML文件，你的目录结构可能如下：
```
src/
+- main/
	+- java/
	| 	+ <source code>
	+-	resources/
			+- public/
				+- error/
				| 	+- 404.html
				+- <other public assets
```

使用FreeMarker模板映射所有5xx错误，你需要如下的目录结构：  

```
src/
	+- main/
		+- java/
		|+	 <source code>
		+-	 resources/
			+- templates/
				+- error/
				| 	+- 5xx.ftl
				+- <other templates>
```

对于更复杂的映射，你可以添加实现ErrorViewResolver接口的beans：  
```java
public class MyErrorViewResolver implements ErrorViewResolver {
@Override
public ModelAndView resolveErrorView(HttpServletRequest request,
	HttpStatus status, Map<String, Object> model) {
		// Use the request or status to optionally return a ModelAndView
		return ...
	}
}
```

你也可以使用Spring MVC特性，比如@ExceptionHandler方法和@ControllerAdvice，ErrorController将处理所有未处理的异常.  

#### Mapping Error Pages outside of Spring MVC
对于不使用Spring MVC的应用，你可以通过ErrorPageRegistrar接口直接注册ErrorPages。该抽象直接工作于底层内嵌servlet容器，即使你没有Spring MVC的DispatcherServlet，它们仍旧可以工作。  
```java
@Bean
public ErrorPageRegistrar errorPageRegistrar(){
	return new MyErrorPageRegistrar();
}
// ...
private static class MyErrorPageRegistrar implements ErrorPageRegistrar {
	@Override
	public void registerErrorPages(ErrorPageRegistry registry) {
		registry.addErrorPages(new ErrorPage(HttpStatus.BAD_REQUEST, "/400"));
	}
}
```

>**Note**  
> 
>注.如果你注册一个ErrorPage，该页面需要被一个Filter处理（在一些非Spring web框架中很常见，比如Jersey，Wicket），那么该Filter需要明确注册为一个ERROR分发器（dispatcher），例如：  

```java
@Bean
public FilterRegistrationBean myFilter() {
	FilterRegistrationBean registration = new FilterRegistrationBean();
	registration.setFilter(new MyFilter());
	...
	registration.setDispatcherTypes(EnumSet.allOf(DispatcherType.class));
	return registration;
}
```

默认的FilterRegistrationBean不包含ERROR dispatcher类型）。  

当部署到一个servlet容器时，Spring Boot通过它的错误页面过滤器将带有错误状态的请求转发到恰当的错误页面。request只有在response还没提交时才能转发（forwarded）到正确的错误页面，而WebSphere应用服务器8.0及后续版本默认情况会在servlet方法成功执行后提交response，你需要设置`com.ibm.ws.webcontainer.invokeFlushAfterService`属性为false来关闭该行为。  

### Spring HATEOAS

如果你开发基于超媒体的 RESTful API，Spring Boot会为Spring HATEOAS 提供自动配置，这在大多数应用中都运作良好。自动配置取代了\@EnableHypermediaSupport，只需注册一定数量的beans就能轻松构建基于超媒体的应用，这些beans包括LinkDiscoverers（客户端支持），ObjectMapper（用于将响应编排为想要的形式）。ObjectMapper可以根据spring.jackson.*属性或`Jackson2ObjectMapperBuilder bean`进行自定义。  
通过注解\@EnableHypermediaSupport，你可以控制Spring HATEOAS的配置，但这会禁用上述ObjectMapper的自定义功能。

### CORS Support

跨域资源共享（CORS）是一个大多数浏览器都实现了的W3C标准，它允许你以灵活的方式指定跨域请求如何被授权，而不是采用那些不安全，性能低的方式，比如IFRAME或JSONP。  

从4.2版本开始，Spring MVC支持CORS。不用添加任何特殊配置，只需要在Spring Boot应用的controller方法上注解@CrossOrigin，并添加CORS配置。通过注册一个自定义addCorsMappings(CorsRegistry)方法的WebMvcConfigurer bean可以指定全局CORS配置：
```java
@Configuration
public class MyConfiguration {
@Bean
public WebMvcConfigurer corsConfigurer() {
	return new WebMvcConfigurer() {
		@Override
		public void addCorsMappings(CorsRegistry registry) {
				registry.addMapping("/api/**");
			}
		};
	}
}
```