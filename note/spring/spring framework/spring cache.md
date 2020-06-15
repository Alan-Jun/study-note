> 除此之外 还可以参考一下 https://blog.csdn.net/luo609630199/article/details/82083593

#### Spring Cache注解

##### 1. 简介

Spring Cache是Spring3.1之后引入的基于注解的缓存方案，旨在通过少量的注解，达到能完成缓存功能的效果。其中涉及到的注解如下：

- **EnableCaching**
- **Cacheable**
- **CacheEvict**
- **CachePut**
- **CacheConfig**
- **Caching**

  目前版本包含了这六个注解，接下来我们将挨个简单介绍下这些注解，然后提供一些简单的例子来作为参考：

> 1. 首先，这些注解都位于`spring-context`包下，具体地址是`org.springframework.cache.annotation`包下，我们通过API学习的时候可以根据这个地址进行查找。
> 2. 其次，Spring Cache的XML实现，是通过命名空间cache来实现的，如果要基于XML来使用的话，记得先引入cache命名空间，接下来我们介绍各个注解的时候，会顺便提一下对应的XML的配置。

##### 2. EnableCaching注解

  该注解是最基础的注解，用于启用Spring的缓存管理功能，类似于Spring XML配置中的<cache:annotation-driven> 标签。开启Spring的缓存功能后，我们还需要配置CacheManager对象。
  CacheManager是用于管理spring cache的实现的，只要使用spring cache，该对象是必须要配置的。该接口有多种实现方式，比如 CaffeineCacheManager，EhCacheCacheManager，ConcurrentMapCacheManager，SimpleCacheManager等；我们来看一个简单的配置：



```java
@Configuration
@EnableCaching
public class AppConfig {
    @Bean
    public CacheManager cacheManager() {
        // configure and return an implementation of Spring's CacheManager SPI
        SimpleCacheManager cacheManager = new SimpleCacheManager();
        cacheManager.setCaches(Arrays.asList(new ConcurrentMapCache("default")));
        return cacheManager;
    }
}
```

不过我们也可以继承CachingConfigurerSupport来配置：



```java
@Configuration
@EnableCaching
public class AppConfig extends CachingConfigurerSupport {

    @Bean
    @Override
    public CacheManager cacheManager() {
        // configure and return an implementation of Spring's CacheManager SPI
        SimpleCacheManager cacheManager = new SimpleCacheManager();
        cacheManager.setCaches(Arrays.asList(new ConcurrentMapCache("default")));
        return cacheManager;
    }

    @Bean
    @Override
    public KeyGenerator keyGenerator() {
        // configure and return an implementation of Spring's KeyGenerator SPI
        return new MyKeyGenerator();
    }
}
```

而EnableCaching注解，有以下三个参数：**很重要，请仔细阅读**

> 1. **proxyTargetClass**，是否使用CGLIB代理，而不是默认的Java接口代理。默认是false；
> 2. **mode**，缓存的代理模式，默认是AdviceMode.PROXY，有两种选择，另一种是AdviceMode.ASPECTJ。使用proxy代理的话，有一个问题就是，缓存方法在外部调用的时候才会被拦截生效，而同一个类的本地调用不会被拦截；这个问题适用于所有使用spring代理的情况，比如Transactional, Async等相关注解；并且proxy模式下，只有public方法上的才会生效；
> 3. **order**，缓存的顺序；

因为Spring Cache是基于Spring的代理模式来实现的，所以上面存在的内部调用问题还是要注意下。

##### 3. Cacheable注解

该注解用于创建缓存，同样来看一下它的参数。

> 1. **value/cacheNames**，这两个参数用于表示缓存的名称，数组类型，可以指定多个；
> 2. **condition**，SpEL 表达式，用于条件过滤，表示当满足该条件的情况下才进行缓存；true或者false，只有为true时才进行缓存；

> 1. **key**，SpEL表达式，缓存的key，Spring提供了专用的缓存相关元数据root对象和result：

| 元数据      | 对应的描述                                                   | 示例                              |
| ----------- | ------------------------------------------------------------ | --------------------------------- |
| methodName  | 当前方法名                                                   | `#root.methodName`                |
| method      | 当前方法                                                     | `#root.method.name`               |
| target      | 当前被调用的对象                                             | `#root.target`                    |
| targetClass | 当前被调用的对象的class                                      | `#root.targetClass`               |
| args        | 当前方法参数组成的数组                                       | `#root.args[0]`                   |
| caches      | 当前被调用的方法使用的Cache                                  | `#root.caches[0].name`            |
| 参数名      | 方法参数的名字，可以直接 #参数名或者使用 #p0或#a0 的 形式， 0代表参数的索引； | `#iban`, `#a0`, `#p0`, `#p<#arg>` |
| result      | 方法执行完成的返回值（仅当方法执行之后的判断有效，如 `unless`, `beforeInvocation=false`） | `#result`                         |

当然，我们在使用root对象的属性作为key时我们也可以将“#root”省略，因为Spring默认使用的就是root对象的属性。

其中key的生成策略有两种，一种是默认策略，一种是自定义策略，但无论哪种策略，都是借助KeyGenerator生成的，其默认策略如下：

- 1.如果方法没有参数，则key是`SimpleKey.EMPTY`;
- 2.如果只有一个参数的话则使用该参数作为key;
- 3.如果参数多于一个的话，则返回一个包含所有参数的`SimpleKey`；

> The default key generation strategy changed with the release of Spring 4.0. Earlier versions of Spring used a key generation strategy that, for multiple key parameters, only considered the hashCode() of parameters and not equals(); this could cause unexpected key collisions (see SPR-10237 for background). The new 'SimpleKeyGenerator' uses a compound key for such scenarios.

大概意思是，原先版本默认的生成策略，对于多个参数的情况只考虑了参数的hashcode，而没有考虑equals，这有可能导致意外的键冲突，而新的`SimpleKeyGenerator`则使用了符合键的操作。

如果默认策略满足不了的话，我们可以自定义我们的KeyGenerator，然后指定Spring Cache使用的KeyGenerator为我们自己定义的KeyGenerator即可；



```kotlin
@Cacheable(value = "userCache", key = "#a0")
public User test(String id) {
    System.out.println("-------------------测试缓存方法-------------");
    User user = queryDb(id);
    return user;
}
```

> 1. **keyGenerator**，前面已经说过，key的自定义生成器，如果不想使用key属性，可以自定义key的实现，继承KeyGenerator；该属性和key是互斥的；
> 2. **cacheManager**，cacheManager的bean配置；
> 3. **cacheResolver**，同样，如果不想使用默认的cacheManager的几种实现，也可以通过该属性来自定义实现，该属性与cacheManager互斥；

> 1. **unless**，用于否决缓存的，和condition不同，condition是在方法开始之前判断，而该属性是在方法执行完成之后判断；所以condition不能通过方法返回值来判断，而unless属性可以；方法返回值是通过元数据result来表示的，true时表示不会缓存，为false时表示进行缓存；



```kotlin
@Cacheable(value = "userCache", unless = "#result == null")
public User test(String id) {
    System.out.println("-------------------测试缓存方法-------------");
    User user = queryDb(id);
    return user;
}
```

需要简单注意的是，返回值如果是Optional的话，表示的是实际的对象，而不是包装器对象；同样，key的几种表示方式也适合该属性；

> 1. sync
>
>    ，表示是否异步，默认为false，也就是同步；一般情况下，Spring的Cache的缓存过期之后，这时候如果多个线程同时对某个数据进行访问，会同时去访问数据库，有可能导致数据库的压力顿时增大，所以Spring4.3之后引入了sync注解，当设置它为true时，会将缓存锁定，只有一个线程的请求会去访问数据库，其他线程都会等待直到缓存可用，这个设置可以减少对数据库的瞬间并发访问；
>
>    - 注意，不支持unless；实际上该属性只是一个提示或建议，至于是否支持要看我们的cache provider ；

##### 4. CacheEvict注解

  该注解用于缓存的清除，由于该注解的参数与Cacheable的参数大部分都是相同的，这里来简单介绍下CacheEvict注解独有的两个参数：

> 1. **allEntries**，是否清空所有的缓存内容，默认为false，如果指定为 true，则方法调用后将清空所有缓存；不过不允许在该值设置为true的情况下，再设置key的值；
> 2. **beforeInvocation**，是否在调用该方法之前清空缓存，默认为false；如果为true，在该方法被调用前就清空缓存，不用考虑该方法的执行结果(即不考虑是否抛出异常)；而默认情况下，如果方法执行时发生异常，则不会清除缓存；



```csharp
@CacheEvict(value = "userCache", key = "#a0", allEntries = true)
public User evict(String id) {
    return queryDb(id);
}
```

##### 5. CachePut注解

该注解用于缓存的更新，不过需要注意的是，如果返回值是JDK 8的Optional的类型的话程序会自动处理；参数方面和Cacheable完全一致，就不多说了。

##### 6. CacheConfig注解

  用于类级别的缓存的公用配置。有的时候，一个类中多个缓存可能会有重复的属性，我们可以使用该注解将这些重复的属性提取出来。参数有**cacheNames**、**keyGenerator**、**cacheManager**、**cacheResolver**这几个属性，我们前文已经说过，这里不多说了；

##### 7. Caching注解

用于组合多个缓存的配置，是一个组注解；属性直接看源码就知道了：



```css
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface Caching {

    Cacheable[] cacheable() default {};

    CachePut[] put() default {};

    CacheEvict[] evict() default {};

}};
```

使用的话可以看下：



```css
@Caching(cacheable = {@Cacheable(value = "userCache", key = "#a0", unless = "#"),
      @Cacheable(value = "userCache", key = "#a0", unless = "#")})
            
```

##### 8. 总结

  Spring Cache中涉及到的常用的注解就这几个，其中多个注解的参数还都是相同的，所以学习起来比较简单。

> 1. 虽然这些注解可以用于方法和类上，但一般情况下都是用于方法上；
> 2. 这些中的大部分注解都可以作为元注解，所以我们可以在这些注解基础上自定义我们的注解；

#### 三、Spring Cache的XML配置

虽然本篇文章的重点是介绍注解的，但还是简单来说下XML中cache的配置。Spring的Cache的XML配置，最外层的标签只有两个：

###### 1. <cache:annotation-driven>

该标签我们前文简单说过，用于开启Spring的缓存功能，其中该标签的几个属性和EnableCaching注解的几个参数是相同的：

> 1. **mode**，**proxy-target-class**，**order**，这三个参数EnableCaching注解是相同的；
> 2. **cache-manager**，这个我们也说过了，用于配置缓存的实现CacheManager，**key-generator**，这个则是自定义key的实现；



```csharp
<cache:annotation-driven key-generator="" cache-manager="" order="" proxy-target-class="" 
    mode="proxy"/>
```

###### 2. <cache:advice>

该标签则是用于配置缓存的具体实现，包含创建，清除，更新等对应功能。我们先来看下配置：



```xml
<cache:advice cache-manager="" key-generator="" id="">
    <cache:caching key-generator="" cache-manager="" condition="" cache="" key="" method="">
        <cache:cacheable method="" key="" cache="" condition="" cache-manager="" key-generator="" unless=""/>
        <cache:cache-put method="" key="" cache="" condition="" cache-manager="" key-generator="" unless=""/>
        <cache:cache-evict method="" key="" cache="" condition="" cache-manager=""  key-generator=""  before-invocation="true" all-entries="true"/>
    </cache:caching>
</cache:advice>
```

<cache:advice>的所有配置都在这了，而这些配置属性与注解中的属性是一一对应的，这里就不多说了。

#### 四、Spring Cache 的一些小问题

##### 1. 缓存有效时间问题

  前面学习@Cacheable的使用的时候，没有涉及到有关缓存的有效时间的设置。这是因为Spring Cache自带的默认的基于`ConcurrentHashMap`的CacheManager实现是没有自动过期这一功能的。Spring Cache支持了许多第三方的Cache实现，不同的Cache对过期时间的处理是不一样的，所以我们如果需要实现有效时间的问题，可以采用如下几种方式：

> 1. 以@Cachable为元注解，自定义我们的的注解实现；
> 2. 在@CacheEvict基础上，结合Scheduled注解，通过定时任务的形式来实现，不过要注意多个key的问题；
> 3. 使用第三方Cache，如Redis，EhCache等实现；

比如说：



```cpp
public CacheManager cacheManager() {
    RedisCacheManager redisCacheManager =newRedisCacheManager(redisTemplate());
    redisCacheManager.setTransactionAware(true);
    redisCacheManager.setLoadRemoteCachesOnStartup(true);
    redisCacheManager.setUsePrefix(true);
    //配置缓存的过期时间
    Mapexpires =newHashMap();
    expires.put("token",expiration);
    redisCacheManager.setExpires(expires);
    return redisCacheManager;
}
```



> 作者：骑着乌龟去看海
> 链接：https://www.jianshu.com/p/e51c8f9b7077
> 来源：简书
> 著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

