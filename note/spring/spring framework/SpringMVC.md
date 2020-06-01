# 1. Spring MVC 简介

Spring Web MVC是构建在Servlet API上的Web框架，从一开始就包含在Spring框架中。

与许多其他Web框架一样，Spring MVC是围绕前端控制器模式设计的，其中`DispatcherServlet`提供用于请求处理的共享算法(作为请求处理的分发控制中心)，而实际工作由可配置的代表组件执行。该模型是灵活的，支持不同的工作流程。

# 2. DispatcherServlet

![1542196554947](assets\1542196554947.png)

## HandlerAdapter 接口

spring mvc 中没有定义 `Controller`这样的一个接口`DispatcherServlet` 控制器通过`HandlerAdapter` 适配器来适配将我们的各种`handller`适配成`DispatcherServlet` 能够使用的`handller`。

## HandlerInterceptor 接口

```java
public interface HandlerInterceptor {

    /**
    * 该方法将在请求处理之前进行调用，只有该方法返回true，才会继续执行后续的Interceptor和
    * Controller，当返回值为true 时就会继续调用下一个Interceptor的preHandle 方法，如果
    * 已经是最后一个Interceptor的时候就会是调用当前请求的Controller方法； 
    * @throws Exception in case of errors
    */
   default boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
         throws Exception {

      return true;
   } 

   /**
    * 该方法将在请求处理之后，DispatcherServlet进行视图返回渲染之前进行调用，可以在这个方
    * 法中对Controller 处理之后的ModelAndView 对象进行操作。 
    * @throws Exception in case of errors
    */
   default void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler,
         @Nullable ModelAndView modelAndView) throws Exception {
   }

   /**
    * 该方法也是需要当前对应的Interceptor的preHandle方法的返回值为true时才会执行，该方法
    * 将在整个请求结束之后，也就是在DispatcherServlet 渲染了对应的视图之后执行。用于进行资
    * 源清理，或则做一些别的处理
    * @throws Exception in case of errors
    */
   default void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler,
         @Nullable Exception ex) throws Exception {
   }

}
```

## HandllerMapping 接口

* 帮助`DispatchServlet`找到正确的`Controller`
* 它工作完毕之后，可以得到，一个包装之后的对象（wrap Controller with HandlerIntercepter）的执行是一个调用链。

## ModelAndView 接口

用于view层的对象都会被转换成`ModelAndView`类型的对象

## ViewResolver 接口

帮助`DispatchServlet`找到我们的`view `

还有很多这样的接口具体的看spring官网的spring mvc的 [Special Bean Types](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-servlet-special-bean-types)

# 3. 配置DispatcherServlet

## 3.1 web.xml  

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://java.sun.com/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
        http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
         version="3.0">

    <display-name>spring mvc demo</display-name>

    <!-- 配置spring mvc dispatcher-->
    <servlet>
        <servlet-name>myDispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!-- 配置dispatcher的初始化参数，也就是我们的spring mvc 的主要配置文件读取路径-->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>/WEB-INF/configs/spring/my-dispatcher-servlet.xml</param-value>
        </init-param>
        <!-- 指定 container init 的时候就创建这个 DispatcherServlet -->
        <load-on-startup>1</load-on-startup>
    </servlet>
  	<!-- 配置处理请求映射 -->
    <servlet-mapping>
        <servlet-name>myDispatcher</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
</web-app>
```

## 3.3 java 方式

```java
public class MyWebApplicationInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {


    @Override
    protected Class<?>[] getRootConfigClasses() {
        return null;
    }

    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class[]{AppConfig.class};
    }

    @Override
    protected String[] getServletMappings() {
        return new String[]{"/"};
    }
}
```

当然这种方式你也可以使用`xml`作为 `Spring web application configuration`. example :
需要注意的是这时候使用的是`AbstractDispatcherServletInitializer`类

```java
public class MyWebApplicationInitializer extends AbstractDispatcherServletInitializer  {
		
  	@Override
    protected WebApplicationContext createRootApplicationContext() {
        return null;
    }

    @Override
    protected WebApplicationContext createServletApplicationContext() {
        XmlWebApplicationContext cxt = new XmlWebApplicationContext();
        cxt.setConfigLocation("/WEB-INF/spring/dispatcher-config.xml");
        return cxt;
    }

    @Override
    protected String[] getServletMappings() {
        return new String[] { "/" };
    }
}
```

更多的配置详情可以查看[官网的介绍](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/web.html#mvc-servlet-config)

# 4. Spring web application configuration

在第3节我们介绍了怎么配置`DispatcherServlet`,那么创建它所用到的`Spring web application configuration`配置，将在这里做介绍，同样的我们会介绍基于`xml`以及`annotation`的方式

## 4.1 xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/mvc
       http://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <!-- 激活那些已经在spring容器里注册过的bean（无论是通过xml的方式还是通过package sanning的方式）上面的注解。 -->
    <context:annotation-config/>

    <!-- 配置只搜索  @Controller注解标注的类 -->
    <context:component-scan base-package="com.stu.controller">
        <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>
    <!--
        自动注册DefaultAnnotationHandlerMapping与AnnotationMethodHandlerAdapter 两个bean,是spring MVC为Controllers分发请求所必须的。
    并提供了：数据绑定支持，比如将url的参数直接映射到controller类中的方法的参数上 ），@NumberFormatannotation支持，@DateTimeFormat支
    持，@Valid支持，读写XML的支持（JAXB），读写JSON的支持（Jackson）。我们处理响应ajax请求时，就使用到了对json的支持。
    -->
    <mvc:annotation-driven/>

    <!--配置 ViewResolver
        可以配置多个ViewResolver,
        可以使用他们的属性order来实现多个ViewResolver的排序
     -->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
          <property name="viewClass" value="org.springframework.web.servlet.view.JstlView"/>
        <property name="prefix" value="/WEB-INF/jsp/"/>
        <property name="suffix" value=".jsp"/>
    </bean>

</beans>
```

### `<mvc:annotation-driven/>`

`<mvc:annotation-driven />` 是一种简写形式，完全可以手动配置替代这种简写形式，简写形式可以让初学都快速应用默认配置方案。`<mvc:annotation-driven />` 会自动注册`DefaultAnnotationHandlerMapping`与`AnnotationMethodHandlerAdapter` 两个bean,是`spring MVC`为`Controllers`分发请求所必须的。
并提供了：数据绑定支持，`@NumberFormatannotation`支持，`@DateTimeFormat`支持，@Valid支持，读写XML的支持（JAXB），读写JSON的支持（Jackson）。

我们处理响应ajax请求时，就使用到了对json的支持。	
对action写JUnit单元测试时，要从spring IOC容器中取`DefaultAnnotationHandlerMapping``与AnnotationMethodHandlerAdapter` 两个bean，来完成测试，取的时候要知道是<mvc:annotation-driven />这一句注册的这两个bean。

## 4.2 annotation

```java
@Configuration
@ComponentScan(basePackages = "com.stu.controller")
@EnableWebMvc
public class AppConfig {

    @Bean
    public ViewResolver getViewResolver() {
        InternalResourceViewResolver viewResolver = new InternalResourceViewResolver();
        viewResolver.setViewClass(org.springframework.web.servlet.view.JstlView.class);
        viewResolver.setPrefix("/WEB-INF/jsp/");
        viewResolver.setSuffix(".jsp");
        return viewResolver;
    }

}
```

### @EnableWebMvc

和<mvc:annotation-driven/> 一样的作用

# 5. 静态资源处理

博客原文https://blog.csdn.net/qq_30038111/article/details/79722624

在讲这个之前我们要先知道，我们服务启动之后，文件的编译之后的存放路劲，现在流行的maven项目结构，编译之后文件会存放到，target目录下面（当然可以指定别的）

下面是我的demo项目的文件路径：

![1542684126166](assets\1542684126166.png)



编译之后的路径：

![1542684179443](assets\1542684179443.png)

* `classpath`就是编译之后的`WEB-INF`下面的`classes`这个目录，当然非web项目的`classpath`是`target/classes`这个目录

* `resources` 目录也会被编译到`classpath`下面

在知道这样的目录结构之后我们再来说

```xml
<mvc:resources location="/WEB-INF/img" mapping="/img/**" cache-period="31556926"/>
```

* `location` 属性用来指定对应的resource 路径，所以他也能使用诸如classpath这样的前缀
* `mapping` 属性指的是将这个resource 映射成什么样的url访问路径
  比如按照下文配置之后 就可一提供：`http://localhost:8080/项目名/img/sss.img`,`http://localhost:8080/项目名/js/jquery-min.js` 
* ` cache-period` 这个属性的作用是，指定这个静态资源在浏览器的换存时间，单位是毫秒，目的是充分利用浏览器端的缓存，在输出静态资源时，会根据配置设置好响应报文的 Expires 和 Cache-Control 值。在接受到静态资源的获取请求时，会检查请求头的 Last_modified 值。如果静态资源没有发生变化，直接返回303响应状态码，指示客户端使用浏览器缓存的数据。**这样可以充分的节省带宽，提升程序性能**

不适用xml的时候怎么配置呢？只需要实现对应的这个方法就可以了

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/resources/**")
            .addResourceLocations("/public", "classpath:/static/")
            .setCachePeriod(31556926);
    }
}
```

[更多的内容请看官网的解释](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/web.html#mvc-config-static-resources)

# Spring MVC的更多配置

https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/web.html#mvc-config 

