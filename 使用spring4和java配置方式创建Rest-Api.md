使用spring4和java配置方式创建Rest-Api
===

> 原文地址: [http://www.baeldung.com/building-a-restful-web-service-with-spring-and-java-based-configuration](http://www.baeldung.com/building-a-restful-web-service-with-spring-and-java-based-configuration)

1.综述
---

> 本文演示了怎么使用Spring设置Rest APi——控制器、HTTP响应码，payload编、解码、内容协商等相关配置。

2.理解Spring中的Rest
---

Spring框架支持两种方式创建RESTful风格的服务
* 使用MVC中的`ModelAndView`
* 使用HTTP message converters

`ModelAndView`方案较旧、文档较完善，但它太过繁琐，配置太重。它将严重影响REST，将其拖入旧模型中。
Spring团队意识到这些，并在Spring 3.0后对REST提供了一流的支持。

基于`HttpMessageConverter `和注解的新方案更轻量级并且更容易实现。配置很少，并且提供了很多你期望的、合理的默认设置。
但新方案的相关文档较少。考文献并未明确指出两者区分以及如何取舍，但这就是Spring 3.0之后构建Restful服务应该使用的方式。

3.java配置

```
@Configuration
@EnableWebMvc
public class WebConfig{
   //
}
```

`@EnableWebMvc`注解做了一些很有用的事情：特别是再REST服务下，它会在class路径下检测是否存在
Jackson和JAXB 2,并会自动创建和注册默认的JSON和XML转换器。该注解功能与下面的XML配置相同：

> `<mvc:annotation-driven />`

这很简洁，也适用于多数情况，但它并不完美，当需要更复杂的配置时，删除注解并直接继承`WebMvcConfigurationSupport`

4.测试
---

从Spring 3.1开始，我们获得了对测试的一流支持。

```
@RunWith( SpringJUnit4ClassRunner.class )
@ContextConfiguration(
  classes = { ApplicationConfig.class, PersistenceConfig.class },
  loader = AnnotationConfigContextLoader.class )
public class SpringTest {
 
   @Test
   public void whenSpringContextIsInstantiated_thenNoExceptions(){
      // When
   }
}
```

java 配置类只需简单的通过`ContextConfiguration`注解指定，新的`AnnotationConfigContextLoader`就会加重配置类中的bean。
注意WebConfig的相关配置类并未包含在启用，因为他们需要在Servlet容器中运行。


5.控制器
---

控制器是整个web应用中restful api的核心控件。出于演示目的，本文的控制器是以一个简单的rest资源-Foo来处理的

```java
@Controller
@RequestMapping("/foos")
class FooController {
 
   @Autowired
   private IFooService service;
 
   @RequestMapping(method = RequestMethod.GET)
   @ResponseBody
   public List<Foo> findAll() {
       return service.findAll();
   }
 
   @RequestMapping(value = "/{id}", method = RequestMethod.GET)
   @ResponseBody
   public Foo findOne(@PathVariable("id") Long id) {
       return RestPreconditions.checkFound( service.findOne( id ));
   }
 
   @RequestMapping(method = RequestMethod.POST)
   @ResponseStatus(HttpStatus.CREATED)
   @ResponseBody
   public Long create(@RequestBody Foo resource) {
       Preconditions.checkNotNull(resource);
       return service.create(resource);
   }
 
   @RequestMapping(value = "/{id}", method = RequestMethod.PUT)
   @ResponseStatus(HttpStatus.OK)
   public void update(@PathVariable( "id" ) Long id, @RequestBody Foo resource) {
       Preconditions.checkNotNull(resource);
       RestPreconditions.checkNotNull(service.getById( resource.getId()));
       service.update(resource);
   }
 
   @RequestMapping(value = "/{id}", method = RequestMethod.DELETE)
   @ResponseStatus(HttpStatus.OK)
   public void delete(@PathVariable("id") Long id) {
       service.deleteById(id);
   }
 
}
```

你也许注意到我使用了一个Guava格式的工具类**RestPreconditions**

```java
public class RestPreconditions {
    public static <T> T checkFound(T resource) {
        if (resource == null) {
            throw new MyResourceNotFoundException();
        }
        return resource;
    }
}
```

控制器的类是非public的——因为它根本不需要是

通常，控制器是传递链的最后一环——它从Spring的前端控制器(DispathcerServlet)接收HTTP请求，然后就简单的将其
委托给服务层，如果没有直接的引用和注入需求，我倾向于不把它声明为public类。

请求映射很简单——对应任何控制器，真正的映射路径和Http请求方法，共同决定了请求的目标方法。
`@RequestBody`注解用于将Http body中的数据与方法参数进行绑定，而`@ResponseBody`则对响应
和返回值类型进行绑定。

它们还确保了使用正确的HTTP 转换器对资源进行编码和解码。内容协商主要是使用Accept来选择使用哪个被激活的HTTP
转换器，当然也可以使用其它HTTP头来决定。



6.http响应码
---

HTTP的响应状态码是REST服务里最重要的环节之一，而且它很快变的非常复杂。正确的理解这些可以知道服务被什么中断。

### 6.1 匹配失败的请求

如果Spring MVC收到一个没有映射的请求，它将认为该请求是不被允许的，并给客户端返回`405 METHOD NOT ALLOWED`。
> ps: 404呢~~~

好的实践是当给客户端返回`405`时，还应该包含允许的请求头，以便指出哪些操作是被允许的。这是Spring MVC的标准行为，
不用额外添加任何配置。

### 6.2 匹配成功的请求

对于任意匹配成功的请求，Spring Mvc都认为请求有效，且当没有指定其它状态码的时会响应`200`

因为控制器额外给create、update、delete声明了响应码，故响应码不是默认的200

### 6.3 客户端错误

客户端出现异常时，异常代码将映射到自定义的异常

在web的任意层抛出异常，spring都将确保映射到正确的HTTP状态码

```java
@ResponseStatus(HttpStatus.BAD_REQUEST)
public class BadRequestException extends RuntimeException {
   //
}
@ResponseStatus(HttpStatus.NOT_FOUND)
public class ResourceNotFoundException extends RuntimeException {
   //
}
```

这些异常时REST API的一部分，他们只应该出现在REST的接入层，而非DAO/DAL层。

另外请注意，这些都是运行时异常，而非编译期异常。

### 6.4 使用 `@ExceptionHandler`

另外一个选项是在控制器上添加`@ExceptionHandler`注解去处理自定义异常，但这个注解只能在当前控制器上起作用，这意味
着需要再每个控制器上都定义一遍。

这种情况，在一些很复杂的有很多控制器的应用场景下很快演化出很多问题。

7.添加maven依赖
---

除了变成的Spring WEb依赖外，我们还要添加REST API的编码和解码依赖

```XML

<dependencies>
   <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-databind</artifactId>
      <version>${jackson.version}</version>
   </dependency>
   <dependency>
      <groupId>javax.xml.bind</groupId>
      <artifactId>jaxb-api</artifactId>
      <version>${jaxb-api.version}</version>
      <scope>runtime</scope>
   </dependency>
</dependencies>

<properties>
   <jackson.version>2.4.0</jackson.version>
   <jaxb-api.version>2.2.11</jaxb-api.version>
</properties>
```

这些jar包将用于将rest资源转化为JSOn或XML

8.总结
---

本篇教程介绍了使用Spring 4并基于java配置实现和配置REST服务，同时讨论了HTTP状态码以及内容的编码和解码。

在下一篇文章中，我们将注重于REST APi可读性、内容协商的高级技能，以及资源的其它展现层次、

与往常一样，文中所用的源码可以在[Github](https://github.com/eugenp/tutorials/tree/master/spring-rest-full)上获取。这是一个基于Maven的项目，很容易引入并允许。
