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

6.http响应码
---

###6.1 不匹配的请求

###6.2 有效并匹配的请求

###6.3 客户端错误

###6.4 使用 `@ExceptionHandler`

7.添加maven依赖
---

8.总结
---