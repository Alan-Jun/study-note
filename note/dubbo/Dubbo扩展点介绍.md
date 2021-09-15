# 扩展点加载方式介绍

在介绍dubbo的扩展加载方式前，我们先看看SPI

## SPI （service provider interface）

先说结论：SPI(service provider interface) 由服务方来提供接口的实现

SPI在我理解起来更多是一种流程的控制，有点类似于控制反转感觉，现在我们来看一个图

![image-20210714195959010](/Users/ypp/work_space/study-note/note/dubbo/assets/image-20210714195959010.png)

正常场景下我们是在使用实现方（provider方）的接口以及实现类来完成我们的功能，但是这种调用有的时候是不够灵活，并且扩展性不怎么好，比如数据库的操作，如果按照这样的逻辑来实现的话，mysql 自己实现一套接口定义，pgsql 自己实现一套接口定义，那么在我们的开发过程中，如果需要从pgsql 切换到mysql 由于接口定义的不同，意味着我们的代码改动量可能会很大，但是如果我们将这种流程反转一下，先提供接口的规范定义，SPI(service provider interface) 服务方提供接口的实现，相信大家在java的使用中都发现了，我们的调用都是遵循同一套契约的（接口定义），然后只需要切换供应商的包，即可使用同一个接口规范实现对数据库的访问。

早期的SPI的例子原生的JDBC代码：

```java
public static void main(String[] args) throws Exception {
	// 1.注册驱动
	Class.forName("com.mysql.jdbc.Driver");
	// 2.获取连接对象
	String url = "jdbc:mysql://localhost:3306/itheima";
	Connection conn = DriverManager.getConnection(url, "root", "root");
	// 3.获取执行SQL语句
	Statement stat = conn.createStatement();
	
	Scanner sc = new Scanner(System.in);
	System.out.println("请输入用户名:");
	String user = sc.nextLine();
	System.out.println("请输入密码:");
	String pass = sc.nextLine();
	// 拼写SQL语句
	String sql = "select * from users where username ='"+user+"' and password ='"+pass+"'";
	System.out.println(sql);
	// 4.调用执行者对象方法,执行SQL语句获取结果集
	ResultSet rs = stat.executeQuery(sql);
	// 5.处理结果集
	while (rs.next()) {
		System.out.println(rs.getString("username") + "\t" + rs.getString("password"));
	}
	// 6.关闭资源
	rs.close();
	stat.close();
	conn.close();
}
```
Driver类是在rt.jar 中定义的service 接口规范，不同的数据库厂商，会有他们对应的实现，我们需要使用什么DB，只需要将他们的包引入。然后按照定义好的接口规范使用即可，不需要关心他们的内部代码细节

在此之后 jdk 中还提供了一个 ServiceLoader<T> 的一个类加载器（这个类加载器的源码很简单这里就不细说了，有兴趣的可以自己去看看） 来实现SPI ，你只需要根据他的规定在根目录（Classpath）创建一个META-INF/service目录。然后在下面创建一个以接口全路径命名的文件，在里面写上你的实现类的全限定名就可以了
如果是多个实现，那就分行写（一行写一个实现的全路径名），在使用的时候会一次性加载并且实例化所有实现类，这样做的话资源利用效率存在问题，优点是不需要像JDBC的时候一样使用 Class.forName来初始化类

## Dubbo 的SPI

Dubbo 的扩展点加载从 JDK 标准的 SPI (Service Provider Interface) 扩展点发现机制加强而来。

Dubbo 改进了 JDK 标准的 SPI 的以下问题：

- JDK 标准的 SPI 会一次性实例化扩展点所有实现，如果有扩展实现初始化很耗时，但如果没用上也加载，会很浪费资源。dubbo 提供了懒加载模式真正使用到的时候才会被实例化（newInstance）；
- 如果扩展点加载失败，连扩展点的名称都拿不到了。比如：JDK 标准的 ScriptEngine，通过 `getName()` 获取脚本类型的名称，但如果 RubyScriptEngine 因为所依赖的 jruby.jar 不存在，导致 RubyScriptEngine 类加载失败，这个失败原因被吃掉了，和 ruby 对应不起来，当用户执行 ruby 脚本时，会报不支持 ruby，而不是真正失败的原因。
- 增加了对扩展点 IoC 和 AOP 的支持，一个扩展点可以直接 setter 注入其它扩展点。

在详细讲解之前我想先简单说一下 `ExtensionLoader` 这个类，他可以说是我们整个Dubbo的SPI的核心，它具备了抽象工厂的能力，又具备了像是策略模式一样生产 adpative 实例的能力（java动态生成）,同时还提供了setter注入能力，同时还有使用到责任链模式一样的wrapper设计，所以在我看来 `ExtensionLoader` 就是一个具有丰富多样能力的多功能工厂

```
org.apache.dubbo.common.extension.ExtensionLoader#getExtension(String name) // 更具name获取指定实现类（如果有 wrapper 实现，那么这个实现类是被wrapper实现类代理的实现类）
org.apache.dubbo.common.extension.ExtensionLoader#getAdaptiveExtension() // 获取自适应的实现类Adaptive实现类，这也是一个代理类，原因是为了实现在实际执行的时候根据配置选取用户配置的真正实现类来执行功能
org.apache.dubbo.common.extension.ExtensionLoader#getActivateExtension(URL url, String key, String group)// 该方法是为了获取所有被 @Activate 注解修饰的实现类列表，根据我们的url读取key指定的参数，然后选取在@Activate所指定的group下的所有实现类，主要是用来构建责任链（使用Filter举例的时候我们会看到这个的作用）
```

### 约定：

在扩展类的 jar 包内放置增加扩展点配置文件 `META-INF/dubbo/接口全限定名` 或 `META-INF/dubbo/接口全限定名，内容为：配置名=扩展实现类全限定名`，多个实现类用换行符分隔。

2.7.7 版本还支持 在这两个目录下 做扩展 `META-INF/dubbo/internal/ ，META-INF/dubbo/external/  `

### 扩展点特性

#### 扩展点自动包装（类似AOP的作用）

自动包装扩展点的 Wrapper 类。`ExtensionLoader` 在加载扩展点时，如果加载到的扩展点有拷贝构造函数，则判定为扩展点 Wrapper 类。

Wrapper类内容：

```java
package com.alibaba.xxx;
 
import org.apache.dubbo.rpc.Protocol;
 
public class XxxProtocolWrapper implements Protocol {
    Protocol impl;
 
    public XxxProtocolWrapper(Protocol protocol) { impl = protocol; }
 
    // 接口方法做一个操作后，再调用extension的方法
    public void refer() {
        //... 一些操作
        impl.refer();
        // ... 一些操作
    }
 
    // ...
}
```

Wrapper 类同样实现了扩展点接口，但是 Wrapper 不是扩展点的真正实现。它的用途主要是用于从 `ExtensionLoader` 返回扩展点时，包装在真正的扩展点实现外。即从 `ExtensionLoader` 中返回的实际上是 Wrapper 类的实例，Wrapper 持有了实际的扩展点实现类。

扩展点的 Wrapper 类可以有多个，也可以根据需要新增。

通过 Wrapper 类可以把所有扩展点公共逻辑移至 Wrapper 中。**新加的 Wrapper 在所有的扩展点上添加了逻辑，有些类似 AOP，即 Wrapper 代理了扩展点。**

**关于Wrapper扩展类的加载这个可以读一下 ExtensionLoader#loadClass 这个方法**

```java
private void loadClass(Map<String, Class<?>> extensionClasses, java.net.URL resourceURL, Class<?> clazz, String name,
                       boolean overridden) throws NoSuchMethodException {
    if (!type.isAssignableFrom(clazz)) {
        throw new IllegalStateException("Error occurred when loading extension class (interface: " +
                type + ", class line: " + clazz.getName() + "), class "
                + clazz.getName() + " is not subtype of interface.");
    }
    if (clazz.isAnnotationPresent(Adaptive.class)) {
        cacheAdaptiveClass(clazz, overridden);
    } else if (isWrapperClass(clazz)) {// 如果扩展类是wrapper类型的扩展类
        cacheWrapperClass(clazz);
    } else {
        clazz.getConstructor();
        if (StringUtils.isEmpty(name)) {
            name = findAnnotationName(clazz);
            if (name.length() == 0) {
                throw new IllegalStateException("No such extension name for the class " + clazz.getName() + " in the config " + resourceURL);
            }
        }

        String[] names = NAME_SEPARATOR.split(name);
        if (ArrayUtils.isNotEmpty(names)) {
            cacheActivateClass(clazz, names[0]);
            for (String n : names) {
                cacheName(clazz, n);
                saveInExtensionClass(extensionClasses, clazz, n, overridden);
            }
        }
    }
}
```

#### 扩展点自适应（实现可选配置的关键）

`ExtensionLoader` 注入的依赖扩展点是一个 `Adaptive` 实例，直到扩展点方法执行时才决定调用是哪一个扩展点实现。

Dubbo 使用 URL 对象（包含了Key-Value）传递配置信息。

扩展点方法调用会有URL参数（或是参数有URL成员）这样依赖的扩展点也可以从URL拿到配置信息，所有的扩展点自己定好配置的Key后，配置信息从URL上从最外层传入。URL在配置传递上即是一条总线。这样也就实现了根据配置选择所使用的扩展实现（包括用户自己实现的扩展类）

为了更加清晰的理解这一点，下面我贴一段Dubbo中的Protocl的动态生成后的自适应扩展点代码

```java
import com.alibaba.dubbo.common.extension.ExtensionLoader;

public class Protocol$Adaptive implements com.alibaba.dubbo.rpc.Protocol {

    public com.alibaba.dubbo.rpc.Exporter export(Invoker arg0) throws com.alibaba.dubbo.rpc.RpcException {
        if (arg0 == null) throw new IllegalArgumentException("com.alibaba.dubbo.rpc.Invoker argument == null");
        if (arg0.getUrl() == null)
            throw new IllegalArgumentException("com.alibaba.dubbo.rpc.Invoker argument getUrl() == null");
        com.alibaba.dubbo.common.URL url = arg0.getUrl();
        String extName = (url.getProtocol() == null ? "dubbo" : url.getProtocol());
        if (extName == null)
            throw new IllegalStateException("Fail to get extension(com.alibaba.dubbo.rpc.Protocol) name from url(" + url.toString() + ") use keys([protocol])");
      // 根据配置获取用户配置的实现类
        com.alibaba.dubbo.rpc.Protocol extension = (com.alibaba.dubbo.rpc.Protocol) ExtensionLoader.getExtensionLoader(com.alibaba.dubbo.rpc.Protocol.class).getExtension(extName);
        return extension.export(arg0);
    }

    public void destroy() {
        throw new UnsupportedOperationException("method public abstract void com.alibaba.dubbo.rpc.Protocol.destroy() of interface com.alibaba.dubbo.rpc.Protocol is not adaptive method!");
    }

    public int getDefaultPort() {
        throw new UnsupportedOperationException("method public abstract int com.alibaba.dubbo.rpc.Protocol.getDefaultPort() of interface com.alibaba.dubbo.rpc.Protocol is not adaptive method!");
    }

    public com.alibaba.dubbo.rpc.Invoker refer(java.lang.Class arg0, com.alibaba.dubbo.common.URL arg1) throws com.alibaba.dubbo.rpc.RpcException {
        if (arg1 == null) throw new IllegalArgumentException("url == null");
        com.alibaba.dubbo.common.URL url = arg1;
        String extName = (url.getProtocol() == null ? "dubbo" : url.getProtocol());
        if (extName == null)
            throw new IllegalStateException("Fail to get extension(com.alibaba.dubbo.rpc.Protocol) name from url(" + url.toString() + ") use keys([protocol])");
            // 根据配置获取用户配置的实现类
        com.alibaba.dubbo.rpc.Protocol extension = (com.alibaba.dubbo.rpc.Protocol) ExtensionLoader.getExtensionLoader(com.alibaba.dubbo.rpc.Protocol.class).getExtension(extName);
        return extension.refer(arg0, arg1);
    }
}

```

#### 扩展点自动装配（可以实现对扩展点的注入以及spring管理的bean的注入）

实现的扩展类中可以定义 setter 方法，ExtensionLoader 会为我们进行注入处理主要支持

1. 注入spring管理的类，
2. 以及@SPI管理的扩展点

两者是互斥的，后面的源码阅读中会解释再次之前我们先看一个例子：这个是dubbo 官方文档的例子：

https://dubbo.apache.org/zh/docs/v2.7/dev/spi/#%E6%89%A9%E5%B1%95%E7%82%B9%E8%87%AA%E5%8A%A8%E8%A3%85%E9%85%8D

链接中文章这里提到的疑问基于对adaptive节了解，以及我们自己演示代码debug的时候我们会了解到，setter注入的SPI的实例是一个adaptive自适应代理类



然后我们在看一下我们本地自己实现的一个示例：

```java
public class MyProtocol implements Protocol {

    private Protocol protocol;

    public void setAbcd(Protocol protocol1) {
        this.protocol = protocol1;
    }

    @Override
    public int getDefaultPort() {
        return protocol.getDefaultPort();
    }

    @Override
    public <T> Exporter<T> export(Invoker<T> invoker) throws RpcException {
        return protocol.export(invoker);
    }

    @Override
    public <T> Invoker<T> refer(Class<T> aClass, URL url) throws RpcException {
        return protocol.refer(aClass,url);
    }

    @Override
    public void destroy() {
        protocol.destroy();
    }
}
```

配置文件：`META-INF/dubbo/com.alibaba.dubbo.rpc.Protocol`

```
myProtocol=com.ymj.demo.spi.MyProtocol
```

debug 的 demo启动代码：

```java
public static void main(String[] args) {
        Protocol myProtocol = ExtensionLoader.getExtensionLoader(Protocol.class).getExtension("myProtocol");
    }
```

**源码导读** ExtensionLoader#injectExtension 是这个注入的核心方法

```java
private T injectExtension(T instance) {

        if (objectFactory == null) {
            return instance;
        }

        try {
            for (Method method : instance.getClass().getMethods()) {
                if (!isSetter(method)) {
                    continue;
                }
                /**
                 * Check {@link DisableInject} to see if we need auto injection for this property
                 */
                if (method.getAnnotation(DisableInject.class) != null) {
                    continue;
                }
                Class<?> pt = method.getParameterTypes()[0];
                if (ReflectUtils.isPrimitives(pt)) {
                    continue;
                }

                try {
                  // 通过setter方法，获取需要注入的参数的name
                    String property = getSetterProperty(method);
                  // 从支持的 objectFactory 中加载实例，进行注入
                    Object object = objectFactory.getExtension(pt, property);
                    if (object != null) {
                        method.invoke(instance, object);
                    }
                } catch (Exception e) {
                    logger.error("Failed to inject via method " + method.getName()
                            + " of interface " + type.getName() + ": " + e.getMessage(), e);
                }

            }
        } catch (Exception e) {
            logger.error(e.getMessage(), e);
        }
        return instance;
    }

 /**
     * get properties name for setter, for instance: setVersion, return "version"
     * <p>
     * return "", if setter name with length less than 3
     */
    private String getSetterProperty(Method method) {
        return method.getName().length() > 3 ? method.getName().substring(3, 4).toLowerCase() + method.getName().substring(4) : "";
    }
```

spring 的的bean的注入工厂

```java
public class SpringExtensionFactory implements ExtensionFactory {
    private static final Logger logger = LoggerFactory.getLogger(SpringExtensionFactory.class);

    private static final Set<ApplicationContext> CONTEXTS = new ConcurrentHashSet<ApplicationContext>();

    public static void addApplicationContext(ApplicationContext context) {
        CONTEXTS.add(context);
        if (context instanceof ConfigurableApplicationContext) {
            ((ConfigurableApplicationContext) context).registerShutdownHook();
        }
    }

    public static void removeApplicationContext(ApplicationContext context) {
        CONTEXTS.remove(context);
    }

    public static Set<ApplicationContext> getContexts() {
        return CONTEXTS;
    }

    // currently for test purpose
    public static void clearContexts() {
        CONTEXTS.clear();
    }

    @Override
    @SuppressWarnings("unchecked")
    public <T> T getExtension(Class<T> type, String name) {

        //SPI should be get from SpiExtensionFactory 也就是说 spi的只能从spi的查询
        if (type.isInterface() && type.isAnnotationPresent(SPI.class)) {
            return null;
        }

        for (ApplicationContext context : CONTEXTS) {
            T bean = getOptionalBean(context, name, type);
            if (bean != null) {
                return bean;
            }
        }

        //logger.warn("No spring extension (bean) named:" + name + ", try to find an extension (bean) of type " + type.getName());

        return null;
    }

}
```

spi的扩展注入工厂

```java
public class SpiExtensionFactory implements ExtensionFactory {

    @Override
    public <T> T getExtension(Class<T> type, String name) {
        if (type.isInterface() && type.isAnnotationPresent(SPI.class)) {
            ExtensionLoader<T> loader = ExtensionLoader.getExtensionLoader(type);
          	// 只要这是一个spi的借口，并且具有是实现类，那么就将Adaptive代码实例返回
            if (!loader.getSupportedExtensions().isEmpty()) {
                return loader.getAdaptiveExtension();
            }
        }
        return null;
    }

}
```

#### 扩展点自动激活（主要是用来获取所有被标注了 @Activate 修饰的实现类，用于构造责任链）

主要是在下面方法中处理的

```
org.apache.dubbo.common.extension.ExtensionLoader#getActivateExtension(URL url, String key, String group)// 该方法是为了获取所有被 @Activate 注解修饰的实现类列表，根据我们的url读取key指定的参数，然后选取在@Activate所指定的group下的所有实现类，主要是用来构建责任链（使用Filter举例的时候我们会看到这个的作用）
```

主要是用于后去一组 @Activate 实现类，然后他们将依据 @Activate 中的 order 字段进行排序，值越大排在越后面，为了更形象的理解可以在dubbo启动的时候debug,ProtocolFilterWrapper类的export，以及refer方法，就能看到 @Activate做了什么

# SPI可扩展实现

https://dubbo.apache.org/zh/docs/v2.7/dev/impls/

* [线程池扩展](dubbo扩展实现/线程池扩展)

* filter的扩展 在他dubbo的官网中没有说，这里我们做一个实用性的介绍，利用到的正式我们的@Activate的特性

  这里我们希望实现，provider的借口的黑白名单控制，以及cat埋点功能，利用切面的方式当前也可以实现，但是在获取dubbo上线文的时候可能会没有使用 Filter 这样能简单方便的获取到我们的接口信息，以及调用方信息，下面是式例

  ```java
  @Slf4j
  @Activate(group = {Constants.PROVIDER}, order = 10002)
  public class CommonMethodControlExecutorFilter implements Filter {
      /**
       * 未命中接口白名单code
       */
      public static final String CODE = "-9999999";
      public static final String ABANDONED_CODE = "-9999997";
      public static final String MATCH_BLACK_LIST_CODE = "-9999996";
      public static final String REGEX = ";";
      public static final String APPLICATION = "application";
      public static final String DUBBO_APPLICATION = "dubboApplication";
      public static final String SEPARATOR = "#";
      public static final String BLACK_LIST = "black_list";
      /**
       * 接口黑白名单控制
       */
      private static final Config config = ConfigService.getConfig("service-auth-control");
      /**
       * 接口api调用监控开关
       */
      private static final Config apiPointConfig = ConfigService.getConfig("service-api-point-control");
  
      @Override
      public Result invoke(Invoker<?> invoker, Invocation invocation) throws RpcException {
          String methodName = invocation.getMethodName();
          Class<?> service = invoker.getInterface();
          String interfaceName = service.getName();
          Object[] arguments = invocation.getArguments();
  
          String application = invocation.getAttachment(APPLICATION);
          if (StringUtils.isBlank(application)) {
              application = invocation.getAttachment(DUBBO_APPLICATION);
          }
          // 当接口权限校验开关打开之后，就会对调用服务做校验
          String methodPath = interfaceName + SEPARATOR + methodName;
          Boolean apiPointSwitch = apiPointConfig.getBooleanProperty(methodPath, false);
          if (apiPointSwitch) {
              Cat.logEvent(methodPath, application);
          }
          /**
           * 接口白名单配置
           */
          String methodAuthWhiteAppListStr = config.getProperty(methodPath, "");
          if (StringUtils.isNotBlank(methodAuthWhiteAppListStr)) {
              if (methodAuthWhiteAppListStr.trim().equals("abandoned")) {
                  return new RpcResult(Response.fail(ABANDONED_CODE, "接口已废弃"));
              }
              log.info(" {} white_list appList={} ", methodPath, methodAuthWhiteAppListStr);
              String[] methodAuthAppList = methodAuthWhiteAppListStr.split(REGEX);
              if (methodAuthAppList.length > 0) {
                  boolean pass = false;
                  for (String applicationName : methodAuthAppList) {
                      if (StringUtils.isBlank(applicationName) && StringUtils.isBlank(application)) {
                          pass = true;
                          break;
                      } else if (applicationName.equals(application)) {
                          pass = true;
                          break;
                      }
                  }
                  if (!pass) {
                      log.info(" {} white_list args={} \n application={} \n methodAuthWhiteAppListStr={}", methodPath, arguments, application, methodAuthWhiteAppListStr);
                      return new RpcResult(Response.fail(MATCH_BLACK_LIST_CODE, "接口权限校验失败"));
                  }
              }
          }
          /**
           * 接口黑名单配置
           */
          String blackListKey = methodPath + "_" + BLACK_LIST;
          String methodAuthBlackAppListStr = config.getProperty(blackListKey, "");
          if (StringUtils.isNotBlank(methodAuthBlackAppListStr)) {
              log.info(" {} black_list appList={} ", methodPath, methodAuthBlackAppListStr);
              String[] methodAuthBlackAppList = methodAuthBlackAppListStr.split(REGEX);
              if (methodAuthBlackAppList.length > 0) {
                  boolean filter = false;
                  for (String applicationName : methodAuthBlackAppList) {
                      if (StringUtils.isBlank(applicationName)) {
                          continue;
                      }
                      if (applicationName.equals(application)) {
                          filter = true;
                      }
                  }
                  if (filter) {
                      log.info(" {} black_list args={} \n application={} \n methodAuthBlackAppListStr={}", methodPath, arguments, application, methodAuthBlackAppListStr);
                      return new RpcResult(Response.fail(CODE, "命中接口的服务黑名单"));
                  }
              }
          }
          return invoker.invoke(invocation);
      }
  }
  ```

