使用Spring 5创建WEB应用
===

> 原文地址: [http://www.baeldung.com/bootstraping-a-web-application-with-spring-and-java-based-configuration](http://www.baeldung.com/bootstraping-a-web-application-with-spring-and-java-based-configuration)

1.综述
---

> 本教程演示了怎么使用Spring创建WEB应用，并讨论了如何从XML文件调整到java 而无需迁移整个XML配置。

2.Maven pom.xml
---

```
<project xmlns=...>
   <modelVersion>4.0.0</modelVersion>
   <groupId>org</groupId>
   <artifactId>rest</artifactId>
   <version>0.1.0-SNAPSHOT</version>
   <packaging>war</packaging>

   <dependencies>

      <dependency>
         <groupId>org.springframework</groupId>
         <artifactId>spring-webmvc</artifactId>
         <version>${spring.version}</version>
         <exclusions>
            <exclusion>
               <artifactId>commons-logging</artifactId>
               <groupId>commons-logging</groupId>
            </exclusion>
         </exclusions>
      </dependency>

   </dependencies>

   <build>
      <finalName>rest</finalName>

      <plugins>
         <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.7.0</version>
            <configuration>
               <source>1.8</source>
               <target>1.8</target>
               <encoding>UTF-8</encoding>
            </configuration>
         </plugin>
      </plugins>
   </build>

   <properties>
      <spring.version>5.0.2.RELEASE</spring.version>
   </properties>

</project>
```

3.基于java配置web应用
---

```
@Configuration
@EnableWebMvc
@ComponentScan(basePackages = "org.baeldung")
public class AppConfig{

}
```

首先,`@Configuration`注解是基于java配置Spring的主要组件，它自身被`@Component`注解注释，
这使被注释类成为了一个标准的bean，并可以被组件扫描。

`@Configuration`最主要的作用就是在源码上将类定义为Spring Ioc容器的bean。有关更详细的说明，请参阅官方文档。

`@EnableWebMvc` 用于配置Spring Web Mvc， 该配置启用了`@Controller`、`@RequestMapping`注解，
它等同于XML中的：
> `<mvc:annotation-driven />`

再转到` @ComponentScan`, 它配置了组件扫描指令，等同于XML中的：
> `<context:component-scan base-package="org.baeldung" />`

Spring 3.1中，`@Configuration`在classpath扫描中被默认排除。在Spring 3.1之前，这些类需要明确指出被排除：
> `excludeFilters = { @ComponentScan.Filter( Configuration.class ) }`

被`@Configuration`注释的类不应该被自动发现，因为它们已经被容器使用-允许它们被发现并将其引入至Spring上下文中将导致以下错误：

```
Caused by: org.springframework.context.annotation.ConflictingBeanDefinitionException: Annotation-specified bean name ‘webConfig’ for bean class [org.rest.spring.AppConfig] conflicts with existing, non-compatible bean definition of same name and class [org.rest.spring.AppConfig]
```

3.1 web.xml
---

```
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns=...>

   <context-param>
      <param-name>contextClass</param-name>
      <param-value>
         org.springframework.web.context.support.AnnotationConfigWebApplicationContext
      </param-value>
   </context-param>
   <context-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>org.baeldung</param-value>
   </context-param>
   <listener>
      <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
   </listener>

   <servlet>
      <servlet-name>rest</servlet-name>
      <servlet-class>
         org.springframework.web.servlet.DispatcherServlet
      </servlet-class>
      <init-param>
         <param-name>contextClass</param-name>
         <param-value>
            org.springframework.web.context.support.AnnotationConfigWebApplicationContext
         </param-value>
      </init-param>
      <init-param>
         <param-name>contextConfigLocation</param-name>
         <param-value>org.baeldung.spring</param-value>
      </init-param>
      <load-on-startup>1</load-on-startup>
   </servlet>
   <servlet-mapping>
      <servlet-name>rest</servlet-name>
      <url-pattern>/api/*</url-pattern>
   </servlet-mapping>

   <welcome-file-list>
      <welcome-file />
   </welcome-file-list>

</web-app>
```

首先，使用`AnnotationConfigWebApplicationContext`代替默认的`XmlWebApplicationContext`去定义和配置web应用的根上下文。
新的`AnnotationConfigWebApplicationContext`接收被`@Configuration`注释的类作为容器的配置，并且这是启用java配置上下文所需要的。

与`XmlWebApplicationContext`不同的是，它默认是没有配置类的路径的，所以必须设置`sevlet`的`contextConfigLocation`参数.
这将指出被`@Configuration`注释的配置类所在的 java 包路径，类的全路径也是支持的。

接下来，`DispatcherServlet`也使用相同类型的上下文定义，唯一的区别是它们从不同的路径中加载配置类。

除此之外，基于XML配置的web.xml与基于JAVA配置的web.xml没有区别。

4.总结
---

以上介绍的方案允许将Spring配置平滑的从XML配置迁移到java配置，旧的和新的混合使用。这对于那些老的项目非常重要，
因为它们的内部有大量XML配置文件而无法一次完成迁移。

这种方式，可以每次少量的移植XML中的bean。

在下一篇 Rest With Spring文章中, 我将介绍 设置MVC项目、配置HTTP状态码、palyload转换、内容协商等。

与往常一样，文中所用的源码可以在[Github](https://github.com/eugenp/tutorials/tree/master/spring-rest-full)上获取。这是一个基于Maven的项目，很容易引入并允许。


