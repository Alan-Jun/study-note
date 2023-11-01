# 什么是 spring framework?

spring  是一个被广泛运用的开源的java开发框架，它提供了一个完整的编程或配置的一个现代化的基于java的企业级应用的开发框架

它的核心要素是它提供了一个易用的编程模型，并且它具有很大的弹性（比如我们可以基于配置（XML,properties...）,也可以基于注解等等方式去使用他的IOC功能；在比如它提供了模块化支持，我们需要使用什么功能只需要引入对应依赖就好，比如jdbc,test... , 需要什么功能就可以使用对应的@EnableXXX注解开启对应功能，比如@EnableCaching,@EnableTransactionManagement,@EnableAsync等等），可以帮助你快速开发企业级java应用。

# spring framework有哪些核心模块

spring-context : 事件驱动(比如`ApplicationEvent`)、注解驱动（比如`@ConponentScan`），模块驱动（比如`@EnableCaching`）等 （**其中context模块又是依赖下面的core,beans,aop,expression，来提供这些包提供的功能**）

spring-core：Spring 基础API 模块，如资源管理，泛型处理 等

spring-beans：SpringBean 相关，如依赖查找，依赖注入

spring-aop : SpringAOP 处理，如动态代理，AOP 字节码提升 

spring-expression：Spring 表达式语言模块

# spring framework 的优势和不足？

# 什么是IOC ?

简单地说，IoC 是反转控制，类似于好莱坞原则，主要有依赖查找 和 依赖注入实现

# 依赖查找和依赖注入的区别？

依赖查找是主动或手动的依赖查找方式（拉的动作），通常需要依赖容器或标准API 实现。而依赖注入则是手动或自动依赖绑定的方式（推的动作），无需依赖特定的容器和 API

 ![image-20200614183222534](assets/image-20200614183222534.png)

# Spring 的IoC 容器有什么优势？

* 完成了典型的IoC 管理：依赖查找和依赖注入 
* AOP 抽象 
* 事务抽象 
* 事件机制 
* SPI 扩展 
* 强大的第三方整合 
  * spring Data
    * jdbc
    * jpa
    * LDAP : [Object-Directory Mapping](https://docs.spring.io/spring-ldap/docs/current/reference/#odm).
    * redis
    * mongoDB
    * 等等
  * jms ： java jms 标准消息服务整个
  * spring messaging ： 为了整个第三方消息服务
  * oxm ： 对象和XMl的映射
  * AOP :
  * 等等
* 易测试性 
  
  * spring - test

# BeanFactory 与 FactoryBean

## BeanFactory

BeanFactory，以Factory结尾，表示它是一个工厂(接口)， 它负责生产和管理bean的一个工厂。在Spring中，BeanFactory是工厂的顶层接口，也是IOC容器的核心接口，因此BeanFactory中定义了**管理Bean的通用方法**，如 **getBean** 和 **containsBean** 等，它的职责包括：实例化、定位、配置应用程序中的对象及建立这些对象间的依赖。BeanFactory只是个接口，并不是IOC容器的具体实现，所以Spring容器给出了很多种实现，如 **DefaultListableBeanFactory**、**XmlBeanFactory**、**ApplicationContext**等，其中XmlBeanFactory就是常用的一个，该实现将以XML方式描述组成应用的对象及对象间的依赖关系

**他的使用场景：**

1、从Ioc容器中获取Bean(byName or byType)

2、检索Ioc容器中是否包含指定的Bean 

3、判断Bean是否为单例

```
public interface BeanFactory {
	//对FactoryBean的转义定义，因为如果使用bean的名字检索FactoryBean得到的对象是工厂生成的对象，
	//如果需要得到工厂本身，需要转义
	String FACTORY_BEAN_PREFIX = "&";

	//根据bean的名字，获取在IOC容器中得到bean实例
	Object getBean(String name) throws BeansException;

	//根据bean的名字和Class类型来得到bean实例，增加了类型安全验证机制。
	<T> T getBean(String name, @Nullable Class<T> requiredType) throws BeansException;

	Object getBean(String name, Object... args) throws BeansException;

	<T> T getBean(Class<T> requiredType) throws BeansException;

	<T> T getBean(Class<T> requiredType, Object... args) throws BeansException;

	//提供对bean的检索，看看是否在IOC容器有这个名字的bean
	boolean containsBean(String name);

	//根据bean名字得到bean实例，并同时判断这个bean是不是单例
	boolean isSingleton(String name) throws NoSuchBeanDefinitionException;

	boolean isPrototype(String name) throws NoSuchBeanDefinitionException;

	boolean isTypeMatch(String name, ResolvableType typeToMatch) throws NoSuchBeanDefinitionException;

	boolean isTypeMatch(String name, @Nullable Class<?> typeToMatch) throws NoSuchBeanDefinitionException;

	//得到bean实例的Class类型
	@Nullable
	Class<?> getType(String name) throws NoSuchBeanDefinitionException;

	//得到bean的别名，如果根据别名检索，那么其原名也会被检索出来
	String[] getAliases(String name);
}
```



## FactoryBean

https://zhuanlan.zhihu.com/p/278318209

首先FactoryBean是一个Bean，但又不仅仅是一个Bean，这样听起来矛盾，但为啥又这样说呢？其实在Spring中，所有的Bean都是由BeanFactory（也就是IOC容器）来进行管理的。但对FactoryBean而言，**这个FactoryBean不是简单的Bean，而是一个能生产或者修饰对象生成的工厂Bean,它的实现与设计模式中的工厂模式和修饰器模式类似**

### 为什么会有FactoryBean

一般情况下，Spring通过反射机制利用的class属性指定实现类实例化Bean。至于为什么会有FactoryBean？原因有两个：

1、 在某些情况下，实例化Bean过程比较复杂，如果按照传统的方式，则需要在其中提供大量的配置信息。配置方式的灵活性是受限的，这时采用编码的方式可能会得到一个简单的方案。Spring为此提供了一个org.springframework.bean.factory.FactoryBean的工厂类接口，用户可以通过实现该接口定制实例化Bean的逻辑。FactoryBean接口对于Spring框架来说占用重要的地位，Spring自身就提供了70多个FactoryBean的实现。它们隐藏了实例化一些复杂Bean的细节，给上层应用带来了便利。

2、 由于第三方库不能直接注册到spring容器，于是可以实现org.springframework.bean.factory.FactoryBean接口，然后给出自己对象的实例化代码即可。

```
public interface FactoryBean<T> {
	//从工厂中获取bean【这个方法是FactoryBean的核心】
	@Nullable
	T getObject() throws Exception;
	
	//获取Bean工厂创建的对象的类型【注意这个方法主要作用是：该方法返回的类型是在ioc容器中getbean所匹配的类型】
	@Nullable
	Class<?> getObjectType();
	
	//Bean工厂创建的对象是否是单例模式
	default boolean isSingleton() {
		return true;
	}
}
```



# ObjectFactory

# Spring IOC 容器启动时做了哪些准备

* IoC 配置元信息读取和解析
  * XML
  * annotation
  * properties

* IoC 提供出来的那些扩展点的实现的载入
  * postProcessBeanFactory
  * invokeBeanFactoryPostProcessors
  * registerBeanPostProcessors
* Spring 事件发布、国 际化等，更多答案将在后续专题章节逐一讨论

