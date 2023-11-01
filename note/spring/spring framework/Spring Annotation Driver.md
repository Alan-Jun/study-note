申明

内容来源于spring官网，我只是做一下笔记加深印象以及方便自己产找，以及后续使用中问题的记录。这里不介绍Spring IOC 基础的东西，以及XML的配置这戏额内容官网都有，需要的情趣查看 https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#beans

**还有一些注解会在别的内容中介绍，比如@EnableTransactionManagement（开启基于注解的事务管理）就会在 [Spring Transaction Management](https://github.com/Alan-Jun/study-note/blob/master/spring%20framework/Spring%20Transaction%20Management.md)**

# 1 使用注解定义Bean

## 1.1 使用注解的前置条件：

* 使用`<context:component-scan />`,除了具有`<context:annotation-config>`的功能之外，还可以在指定的package下扫描以及注册javabean 。

  这个标签还有很多别的属性可以指定这里介绍一部分

  * `base-package=""`属性用来指定具体的包路径

  * `scoped-proxy=""` 这个属性指定使用什么样的动态代理方式来实现，取值：
    * `interfaces` : 使用 jdk 的基于接口的动态代理
    * `targetClass` : 使用 cglib的基于类的方法拦截调用的动态代理
  * `name-generator=""`: 用于自定义命名规则的属性，1.4.1 节 会有介绍
  * `use-default-filters=""` : 默认为true , 会扫描指定下的所有被 1.2 节中说所注解修饰的类，并且注册成bean, 指定 false 请看 1.1.1节 的具体说明
  * `resource-pattern="" : 暂时不知道用法`
  * `scope-resolver=""` : 用于自定义Scope策略，具体的看 1.5.1 节

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd">
    <!--<bean id="bean2" class="com.stu.springdemo.annotation.resource.Bean1"/>-->
    
    <context:component-scan base-package="com.stu.springdemo.annotation"/>
</beans>
```

**这个配置也是可以使用注解来替代的，后文1.3介绍的`@Configuration`注解，修饰的类将会是配置类，他的作用和xml基本完全相同，结合一些对应的注解基本可以完全替代xml,比如这里我们用 @ComponentScan(basePackages ="com.stu.springdemo.annotation") 注解可以完全替代```<context:component-scan> ```标签**

* ```<context:annotation-config></context:annotation-config>``` 是用于激活那些已经在spring容器里注册过的bean（无论是通过xml的方式还是通过package sanning的方式）上面的注解。

**总结**：推荐使用上面第一种方式。不推荐使用`<context:annotation-config></context:annotation-config>`

### 1.1.1 使用过滤器进行自定义扫描

`<context:component-scan/>`这个标签`use-default-filters="true"默认为true`,会扫描指定下的所有被 1.2节 中说所注解修饰的类，并且注册成bean，但是有的情况我们需要一些更灵活的使用，那么它有有两个子标签：

* `<context:include-filter type="" expression=""/>`: 用于指定需要过滤后得到的项
* `<context:exclude-filter type="" expression=""/>` :用于指定需要去除的项

| Filter Type | Examples Expression        | Description                                                  |
| :---------: | -------------------------- | ------------------------------------------------------------ |
| annotation  | org.example.SomeAnnotation | 符合SomeAnnoation的target class                              |
| assignable  | org.example.SomeClass      | 指定class或interface的全名                                   |
|   aspectj   | org.example..*Service+     | AspetJ语法                                                   |
|    regex    | org.example.Default.*      | 正则表达式                                                   |
|   custom    | org.example.MyTypeFilter   | Spring3新增自订Type,称作org.springframework.core.type.TypeFilter |

**注意：使用两个子标签，指定`<context:component-scan/>`的`use-default-filters="false"`,才有效果**

## 1.2 作用于类的注解

这类注解的最基础的注解（不考虑spring 对JSR 330支持的情况下 ）是spring的 `@Component` 这个注解（元注解）。然后就是用这个注解修饰的注解了，比如`@Repository`  ，`Service`

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Repository {
    @AliasFor(
        annotation = Component.class
    )
    String value() default "";
}
```

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Service {
    @AliasFor(
        annotation = Component.class
    )
    String value() default "";
}
```

可以看出他们都是使用了`@Component`这个注解（元注解）修饰的注解，所以我才会说`@Component`这是一个最基础的注解

总结一下这样的注解有很多，我先列举一部分我目前知道的，后续会做补充

* `@Component` :作为最基础的注解，也说明他是一个通用的注解，可以用于定义任何的bean
* `@Configuration`: 配合@Bean 注解达到 让 java代码类似XMl配置bean一样的效果
* `@Repository` ：常用于修饰 Dao类(持久层)
* `@Service` ： 常用于修饰 Service类（服务层）
* `@Controller` ：常用于Controller类（控制层）
* 以及各种用`@Component`修饰的注解（包括自定义的注解），相当于他的子注解

## 1.3`@Configuration`+`@Bean` 作用方法的注解 

这个注解需要搭配 `@Configuration`注解使用，当然也可以搭配通用的`@Component`注解使用,效果和XMl中配置的效果类似，但是少了部分功能，这部分功能需要使用 别的注解来补充（这些注解后续会有介绍）

注意：`@Bean`配置的bean 默认单例的，如果需要指定别的请搭配1.5节中的`@Scope`使用

实例：

```java
public interface Animal {
    void eat();
}
```

```java
public class Lion implements Animal{

    @Value("鸡肉")
    private String foodName;

    public void initMethod(){
        System.out.println("I am a small lion");
    }
    public void destroyMethod(){
        System.out.println("I am a small lion,I am hungry");
    }
    @Override
    public void eat() {
        System.out.println("Lion eat "+foodName);
    }
}
```

```java
public class Tiger implements Animal{
    public void initMethod(){
        System.out.println("I am a small tiger");
    }
    @PostConstruct
    public void postConstruct(){
        System.out.println("I am a small tiger,PostConstruct");
    }
    @PreDestroy
    public void preDestroy(){
        System.out.println("I am a small tiger,I am digesting food.");
    }
    public void destroyMethod(){
        System.out.println("I am a small tiger,I am hungry");
    }
    @Override
    public void eat() {
        System.out.println("tiger eat meat");
    }
}
```

```java
@Configuration
public class ConfigyrationDemo {

    @Bean(name="tiger")
    public Tiger getTiger(){
        return new Tiger();
    }

    @Bean(name="lion",initMethod = "initMethod",destroyMethod = "destroyMethod")
    public Lion lion(){
        return new Lion();
    }
}
```

## 1.4 Bean 名称的生成策略

在使用 1.2 中所说的注解的时候如果不给注解显示的指定名称，spring中默认的beanName 的生成器 BeanNamegenerator 会使用类名且类名第一个字母小写作为beanName.比如 类 BeanDemo,它的beanName=beanDemo。

### 1.4.1 自定义命名策略

实现BeanNamegenerator接口，并且包含一个无参数的构造方法。然后通过配置文件：

```xml
<context:component-scan base-package="xxx.xxx.xxx" name-generator="xxx.xxx.MyNamegenerator"/>
```

指定即可，也可以使用`@ComponentScan` 注解

## 1.5 @Scope

[作用域类型](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#beans-factory-scopes)

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Scope {
    @AliasFor("scopeName")
    String value() default "";

    @AliasFor("value")
    String scopeName() default "";

    ScopedProxyMode proxyMode() default ScopedProxyMode.DEFAULT;
}
```

和xml中的作用域相同，通过`@Scope`注解也能达到相同的效果，

**从源码可以看出这个注解可以用于方法和累上面，相对应的就是配合上文的 1.2节 和 1.3节 中的注解使用了，他们默认情况下都是单例的。**

**该注解同时还可以指定使用什么样的代理方式**

```java
@Configuration
public class ConfigyrationDemo {

    @Bean(name="tiger")
    public Tiger getTiger(){
        return new Tiger();
    }

    @Bean(name="lion",initMethod = "initMethod",destroyMethod = "destroyMethod")
    @Scope("prototype")
    public Lion lion(){
        return new Lion();
    }
}
```

```java
@Component
@Scope("prototype")
public class BeanAnnotation {

    public void say(){
        System.out.println(" I am a spring bean by Component Annotation "+this);
    }

    public void myHashCode(){
        System.out.println(this.hashCode());
    }
}
```

### 1.5.1 自定义Scope策略

实现ScopeMetaDataResolver接口，并且包含一个无参数的构造方法。然后通过配置文件：

```xml
<context:component-scan base-package="xxx.xxx.xxx" name-generator="xxx.xxx.MyScopeMetaDataResolverr"/>
```
或则`@ComponentScan`的方式

## 1.6 `@AutoWired `

```java
@Target({ElementType.CONSTRUCTOR, ElementType.METHOD, ElementType.PARAMETER, ElementType.FIELD, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Autowired {
    boolean required() default true;
}
```

**从源码我们可以看出，他可以用于方法，字段，构造方法，参数，以及注解**

### 1.6.1 required attribute

**从源码中我们可以看到，该注解中定义了一个 required属性，默认是true,作用是如果在装配过程中找不到匹配的bean,那么会抛出异常，相当于null值检查**,功能和 `@Required` 这个注解相同

**如果指定成false,那么将不会做这样的null检查，从Spring Framework 5.0开始，您还可以使用`@Nullable`注释（任何包中的任何类型，例如`javax.annotation.Nullable`来自JSR-305），来达到  required = false的效果。**

### 1.6.2 使用方法

同XMl中类似，不过`@AutoWired` 注解是通过byType+ByName.的方式去自动匹配的：

* 作用于字段：首先先通过byType的方式去查找有没有匹配类型的bean,如果只有一个直接匹配，如果有多个，以字段名为name，去匹配对应name的bean,如何指定对应的 name呢？ 看 1.7 节

* 作用于方法：首先方法的参数数量，名称没有要求。匹配方式同上面的类似，不过name改成参数名，如何指定对应的 name呢？ 看 1.7 节

* 作用于构造方法：如果存在多个带参数的构造方法，那么该注解只能用于一个构造方法之上，匹配方式同上

* 也可以混合使用

* 特殊的用法：

  * 您还可以使用`@Autowired` 修饰那些众所周知的解析依赖接口：`BeanFactory`，`ApplicationContext`，`Environment`，`ResourceLoader`，`ApplicationEventPublisher`，和`MessageSource`。这些接口及其扩展接口（如`ConfigurableApplicationContext`或`ResourcePatternResolver`）会自动解析，无需特殊设置。 

    **由于`@AutoWired`是由 `BeanpostProcessor` 处理的,所以该注解不能用于 `BeanpostProcessor`和`BeanFactoryPostProcessor`类型的字段,他们必须适使用`XMl`或则`@Bean`注解加载**.

  * 通过将注释添加到容器中存在的特定类型的bean的数组（或集合）的字段或方法，也可以得到当前容器中所有这样类型的bean , 例子： 

    ```java
    interface Person {
        void say();
    }
    ```

    ```java
    @Component
    public class Men implements Person {
        @Override
        public void say() {
            System.out.println(" i am men");
        }
    }
    ```

    ```java
    @Component
    public class Women implements Person {
        @Override
        public void say() {
            System.out.println(" i am women");
        }
    }
    ```

    ```java
    @Component
    public class AutoWiredDemo {
    
        private List<Person> persons;
    	@Autowired
    	private Map<String,Person> personMap;
    	
    	@Autowired
      public void setPersons(List<Person> persons) {
          this.persons = persons;
      }
    }
    ```
    **如果希望按特定顺序对数组或列表中的项进行排序，则目标bean可以实现`org.springframework.core.Ordered`接口或使用`@Order`或标准`@Priority`注释。否则，它们的顺序将遵循容器中相应目标bean定义的注册顺序。** 

## 1.7 @Qualifier help found by beanName

```java
interface Person {
    void say();
}
```

```java
@Component
public class Men implements Person {
    @Override
    public void say() {
        System.out.println(" i am men");
    }
}
```

```java
@Component
public class Women implements Person {
    @Override
    public void say() {
        System.out.println(" i am women");
    }
}
```

```java
@Component
public class AutoWiredDemo {
    
	@Autowired
    @Qualifier("men")
    private Person person;
    
}

```
```java
@Component
public class AutoWiredDemo {
    
    private Person person;
	
   	@Autowired
    public void setPersons( @Qualifier("men")Person person) {
        this.person = person;
    }
}
```



## 1.8 导入外部资源

## 1.8.1 @ImportResource

使用` @ImportResource`注解配合 `@Value`注解，可以实现读取资源文件中的key对应的value值

```java
@value("${key:default")
private String var;
```

以上声明指导spring根据key去属性配置文件查找value，如果没找到，则使用default作为默认值。

**特别注意：这里我们代码中使用key，不是 username,因为如果在spring中使用`$(username)`,获取到的将是当前系统的用户名，而不是资源文件中配置的username.**

```java
@Configuration
@ImportResource("classpath:spring/import/importResourceDemo.xml")
public class ImportResourceDemo {
    @Value("${jdbc.username}")
    private String username;

    @Value("${jdbc.password}")
    private String password;

    @Value("${jdbc.url}")
    private String url;

    @Bean(name = "myDriverManager")
    @Scope(value = "prototype", proxyMode = ScopedProxyMode.TARGET_CLASS)
    public MyDriverManager getMyDriverManager() {
        return new MyDriverManager(username, password, url);
    }
}
```



```java
public class MyDriverManager {

    public MyDriverManager(String username, String password, String url) {
        System.out.println(username);
        System.out.println(password);
        System.out.println(url);
    }
}
```

importResourceDemo.xml

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
               xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
               xmlns:context="http://www.springframework.org/schema/context"
               xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd">
<!--<bean id="bean2" class="com.stu.springdemo.annotation.resource.Bean1"/>-->
<context:property-placeholder location="classpath:spring/import/jdbc.properties"/>
</beans>
```

jdbc.properties

```properties
jdbc.username=root
jdbc.password=116611 
jdbc.url=jdbc:mysql://localhost:3306/train?useUnicode=true&amp;characterEncoding=UTF-8
```

## 1.8.2 @PropertySource

用法和上面的一样，不过他是直接针对的 properties文件，这样就不用写上面的 importResourceDemo.xml 文件了

这个properties文件会加载到Spring的`Environment`中

[官网详解](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#beans-environment)

阿里云中的一些论坛博客中关于@PropertySource的解释：

https://yq.aliyun.com/articles/371526?spm=5176.11065265.1996646101.searchclickresult.21f3500eaqlVhB

https://www.aliyun.com/jiaocheng/771421.html



## 1.9 JSR 250

### 1.9.1 @Resourc

和`@Autowired`使用基本方式基本相同,那么他们不同点是:

* `@Resource` 一般按照ByName匹配,别的方式比较特殊,也不推荐使用,这里就不介绍,有兴趣可以自己写demo总结一下.

* `@Resource`只能用于单值自动注入(也就是在修饰方法时,方法只能是单个参数)
* `@Resource` 可以指定name属性,明确匹配对应的bean

ByName匹配规则:

* 作用于字段: 不明确指定注解的name属性的时候,默认按照字段的名去匹配,明确指定name那么按照.....
* 作用于方法: 不明确指定注解的name属性的时候,默认按照方法参数名去匹配,明确指定name那么按照.....
* 作用于构造方法:同上

### 1.9.2 @PostConstruct 和 @PreDestroy

这个注解是spring Bean 生命周期 中初始化和销毁的注解实现机制。

如果为bean配置了多个生命周期机制，并且每个机制都配置了不同的方法名称，则每个配置的方法都按照下面列出的顺序执行。但是，如果`init()`为多个生命周期机制配置了相同的方法名称（例如， 对于初始化方法），则该方法将执行一次。

为同一个bean配置的多个生命周期机制，具有不同的初始化方法，执行顺序如下所示：

- 用注释方法注释 `@PostConstruct`
- `afterPropertiesSet()`由`InitializingBean`回调接口定义
- 自定义配置的`init()`方法

Destroy方法以相同的顺序调用：

- 用注释方法注释 `@PreDestroy`
- `destroy()`由`DisposableBean`回调接口定义
- 自定义配置的`destroy()`方法

## 2.0 JSR 330

从spring 3.0 开始支持，使用前需要导入依赖，这里通过maven的方式导入

```xml
<dependency>
    <groupId>javax.inject</groupId>
    <artifactId>javax.inject</artifactId>
    <version>1</version>
</dependency>
```

### 2.1 @Inject

等效与`@AutoWired`，不过由于它没有 required 属性，所以要搭配 `@Nullable`才能实现`@AutoWired(required = false)`这样的效果	

### 2.2 @Named

```xml
@Qualifier
@Documented
@Retention(RetentionPolicy.RUNTIME)
public @interface Named {
    String value() default "";
}
```

修饰类相当于`@Component`,除此之外和`@Qualifier`用法相似(从源码也能看出来)