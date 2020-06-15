# 代理

代理就是要构造一个和被代理类具有同样行为的对象，我们有 静态代理和动态代理

# 动态代理

在java中，动态代理有两种实现方式

* [一种是基于jdk的动态代理](#jdk动态代理)

  它的实现方式是基于接口，以及需要代理的类，生成动态代理类，最终生成的代理类持有我们的 handler,handler中持有被代理对象，代理类中生成了和被代理类同样的处理方法（和被代理类具有同样行为），里面保存了被代理类的所有方法的反射 Method 对象，这样就实现了动态代理，并对原来的方法进行增强

* [一种是基于cglib的动态代理](#cglib动态代理)

  它是另一种思想，通过继承的方式来达到目的。子类重写父类（被代理类）的方法，并可以对其进行增强，来实现动态代理

**jdk 1.6 , 1.7 以前 jdk的动态代理的性能是要低于 cglib的， 但是1.8开始性能已经大于等于cglib了。现在的使用主要是根据场景不同做选用**

# jdk动态代理

我们知道JDK的实现方式是基于接口，以及需要代理的类，生成动态代理类

## 使用demo

所以我们首先需要定义一个接口，随便写一个实现类

```java
public interface ByeInterface {
    void sayBye();
}
// 接口实现类
public class Bye implements ByeInterface {
    @Override
    public void sayBye() {
        System.out.println("Bye Evan!");
    }
}
```

然后定义一个handler，这里面是完成的是最终在方法调用时候执行方法调用的前后处理逻辑

```kotlin
public class ProxyHandler implements InvocationHandler{
    private Object object;
    public ProxyHandler(Object object){
        this.object = object;
    }
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("Before invoke "  + method.getName());
        method.invoke(object, args);// 这里通过反射调用你的实际的被代理的类 proxy的方法
        System.out.println("After invoke " + method.getName());
        return null;
    }
}
```

执行动态代理

```java
public static void main(String[] args) {
    System.getProperties().setProperty("sun.misc.ProxyGenerator.saveGeneratedFiles", "true");
    
    ByeInterface hello = new Bye();
    
    InvocationHandler handler = new ProxyHandler(hello);
    
    HelloInterface proxyHello = (HelloInterface) Proxy.newProxyInstance(hello.getClass().getClassLoader(), hello.getClass().getInterfaces(), handler);

    proxyHello.sayHello();
}
```

输出

    输出：
    Before invoke sayBye
    Bye Evan!
    After invoke sayBye
## 原理

动态代理具体步骤：

1. 通过实现 InvocationHandler 接口创建自己的调用处理器；
2. 通过为 Proxy 类指定 ClassLoader 对象和一组 interface 来创建动态代理类；
3. 通过反射机制获得动态代理类的构造函数，其唯一参数类型是调用处理器接口类型；
4. 通过构造函数创建动态代理类实例，构造时调用处理器对象作为参数被传入。

我们看一下 newProxyInstance 都做了些什么吧

### newProxyInstance 

```java
public static Object newProxyInstance(ClassLoader loader,
                                      Class<?>[] interfaces,
                                      InvocationHandler h)
    throws IllegalArgumentException
{
    Objects.requireNonNull(h);

    final Class<?>[] intfs = interfaces.clone();
    final SecurityManager sm = System.getSecurityManager();
    if (sm != null) {
        checkProxyAccess(Reflection.getCallerClass(), loader, intfs);
    }

    /*
     * Look up or generate the designated proxy class.
     * 生成代理类 生成方法看 ProxyClassFactory 的apply方法
     */
    Class<?> cl = getProxyClass0(loader, intfs);

    /*
     * Invoke its constructor with the designated invocation handler.
     */
    try {
        if (sm != null) {
            checkNewProxyPermission(Reflection.getCallerClass(), cl);
        }
		// 获取代理类的构造方法 这个我们需要看 下面Proxy类的带参数的那个构造方法 
        final Constructor<?> cons = cl.getConstructor(constructorParams);
        final InvocationHandler ih = h;
        // 如果构造参数不是 public 通过 setAccessible 支持访问
        if (!Modifier.isPublic(cl.getModifiers())) {
            AccessController.doPrivileged(new PrivilegedAction<Void>() {
                public Void run() {
                    cons.setAccessible(true);
                    return null;
                }
            });
        }
        // 生成代理类了，将我们些的 handler 传了过去
        return cons.newInstance(new Object[]{h});
    } catch (IllegalAccessException|InstantiationException e) {
        throw new InternalError(e.toString(), e);
    } catch (InvocationTargetException e) {
        Throwable t = e.getCause();
        if (t instanceof RuntimeException) {
            throw (RuntimeException) t;
        } else {
            throw new InternalError(t.toString(), t);
        }
    } catch (NoSuchMethodException e) {
        throw new InternalError(e.toString(), e);
    }
}
```

### ProxyClassFactory

```java
private static final class ProxyClassFactory
    implements BiFunction<ClassLoader, Class<?>[], Class<?>>
{
    // prefix for all proxy class names
    private static final String proxyClassNamePrefix = "$Proxy";

    // next number to use for generation of unique proxy class names
    private static final AtomicLong nextUniqueNumber = new AtomicLong();

    @Override
    public Class<?> apply(ClassLoader loader, Class<?>[] interfaces) {

        Map<Class<?>, Boolean> interfaceSet = new IdentityHashMap<>(interfaces.length);
        for (Class<?> intf : interfaces) {
      
            Class<?> interfaceClass = null;
            try {
                interfaceClass = Class.forName(intf.getName(), false, loader);
            } catch (ClassNotFoundException e) {
            }
            if (interfaceClass != intf) {
                throw new IllegalArgumentException(
                    intf + " is not visible from class loader");
            }
            /*
             * Verify that the Class object actually represents an
             * interface.
             */
            if (!interfaceClass.isInterface()) {
                throw new IllegalArgumentException(
                    interfaceClass.getName() + " is not an interface");
            }
            /*
             * Verify that this interface is not a duplicate.
             */
            if (interfaceSet.put(interfaceClass, Boolean.TRUE) != null) {
                throw new IllegalArgumentException(
                    "repeated interface: " + interfaceClass.getName());
            }
        }
		// 上面的代码主要是做接口的校验，看看是不是都是接口类
        
        
        String proxyPkg = null;     // package to define proxy class in
        int accessFlags = Modifier.PUBLIC | Modifier.FINAL;

        /*
         * 拿到代理类的 package
         * Record the package of a non-public proxy interface so that the
         * proxy class will be defined in the same package.  Verify that
         * all non-public proxy interfaces are in the same package.
         */
        for (Class<?> intf : interfaces) {
            int flags = intf.getModifiers();// 获取接口的 访问权限
            if (!Modifier.isPublic(flags)) {// 非public 的接口会走到这里面，我们日常使用的时候几倍呢不会走到这里面的
                accessFlags = Modifier.FINAL;
                String name = intf.getName();
                int n = name.lastIndexOf('.');
                String pkg = ((n == -1) ? "" : name.substring(0, n + 1));
                if (proxyPkg == null) {
                    proxyPkg = pkg;
                } else if (!pkg.equals(proxyPkg)) {
                    throw new IllegalArgumentException(
                        "non-public interfaces from different packages");
                }
            }
        }

        if (proxyPkg == null) {
            // if no non-public proxy interfaces, use com.sun.proxy package
            proxyPkg = ReflectUtil.PROXY_PACKAGE + ".";
        }

        /*
         * Choose a name for the proxy class to generate.
         */
        long num = nextUniqueNumber.getAndIncrement();
        // 通常我们的使用时候 胜场的代理类名都是 com.sun.proxy.$Proxy[num] 这样的代理类
        String proxyName = proxyPkg + proxyClassNamePrefix + num;

        /*
         * Generate the specified proxy class.
         * 生成指定代理类的字节码文件， 这部分代码看下面的 Pro
         */
        byte[] proxyClassFile = ProxyGenerator.generateProxyClass(
            proxyName, interfaces, accessFlags);
        try {
            return defineClass0(loader, proxyName,
                                proxyClassFile, 0, proxyClassFile.length);//生成代理类Class
        } catch (ClassFormatError e) {
            /*
             * A ClassFormatError here means that (barring bugs in the
             * proxy class generation code) there was some other
             * invalid aspect of the arguments supplied to the proxy
             * class creation (such as virtual machine limitations
             * exceeded).
             */
            throw new IllegalArgumentException(e.toString());
        }
    }
}
```

### Proxy

```java
public class Proxy implements java.io.Serializable {

    private static final long serialVersionUID = -2222568056686623797L;

    /** parameter types of a proxy class constructor */
    private static final Class<?>[] constructorParams =
        { InvocationHandler.class };

    /**
     * a cache of proxy classes
     */
    private static final WeakCache<ClassLoader, Class<?>[], Class<?>>
        proxyClassCache = new WeakCache<>(new KeyFactory(), new ProxyClassFactory());

    /**
     * the invocation handler for this proxy instance.
     * @serial
     */
    protected InvocationHandler h;

    /**
     * Prohibits instantiation.
     */
    private Proxy() {
    }
	// 
    protected Proxy(InvocationHandler h) {
        Objects.requireNonNull(h);
        this.h = h;
    }
    
    // 省略部分代码
}
```

### ProxyGenerator

```java
private static final boolean saveGeneratedFiles = (Boolean)AccessController.doPrivileged(new GetBooleanAction("sun.misc.ProxyGenerator.saveGeneratedFiles"));

public static byte[] generateProxyClass(final String var0, Class<?>[] var1, int var2) {
    ProxyGenerator var3 = new ProxyGenerator(var0, var1, var2);
    // 真正用来生成代理类字节码文件的方法在这里
    final byte[] var4 = var3.generateClassFile();
    if (saveGeneratedFiles) {// 是否将JDK动态代理生成的class文件保存到本地
        AccessController.doPrivileged(new PrivilegedAction<Void>() {
            public Void run() {
                try {
                    int var1 = var0.lastIndexOf(46);
                    Path var2;
                    if (var1 > 0) {
                        Path var3 = Paths.get(var0.substring(0, var1).replace('.', File.separatorChar));
                        Files.createDirectories(var3);// 创建这样一个文件夹
                        var2 = var3.resolve(var0.substring(var1 + 1, var0.length()) + ".class");
                    } else {
                        var2 = Paths.get(var0 + ".class");
                    }

                    Files.write(var2, var4, new OpenOption[0]);// 写文件
                    return null;
                } catch (IOException var4x) {
                    throw new InternalError("I/O exception saving generated file: " + var4x);
                }
            }
        });
    }

    return var4;
}

// 实际的生成字节码的代码
private byte[] generateClassFile() {
    //下面一系列的addProxyMethod方法是将接口中的方法和Object中的方法添加到代理类的代理方法中(proxyMethod)
    this.addProxyMethod(hashCodeMethod, Object.class);
    this.addProxyMethod(equalsMethod, Object.class);
    this.addProxyMethod(toStringMethod, Object.class);
    Class[] var1 = this.interfaces;
    int var2 = var1.length;

    int var3;
    Class var4;
    //获得所有接口的所有方法并添加到代理方法中
    for(var3 = 0; var3 < var2; ++var3) {
        var4 = var1[var3];
        Method[] var5 = var4.getMethods();
        int var6 = var5.length;

        for(int var7 = 0; var7 < var6; ++var7) {
            Method var8 = var5[var7];
            this.addProxyMethod(var8, var4);
        }
    }
	
    Iterator var11 = this.proxyMethods.values().iterator();

    List var12;
    while(var11.hasNext()) {
        var12 = (List)var11.next();
        checkReturnTypes(var12);
    }

    Iterator var15;
    try {
        //生成代理类的构造函数
        this.methods.add(this.generateConstructor());
        var11 = this.proxyMethods.values().iterator();

        while(var11.hasNext()) {
            var12 = (List)var11.next();
            var15 = var12.iterator();

            while(var15.hasNext()) {
                ProxyGenerator.ProxyMethod var16 = (ProxyGenerator.ProxyMethod)var15.next();
                this.fields.add(new ProxyGenerator.FieldInfo(var16.methodFieldName, "Ljava/lang/reflect/Method;", 10));
                this.methods.add(var16.generateMethod());
            }
        }

        this.methods.add(this.generateStaticInitializer());
    } catch (IOException var10) {
        throw new InternalError("unexpected I/O Exception", var10);
    }

    if (this.methods.size() > 65535) {
        throw new IllegalArgumentException("method limit exceeded");
    } else if (this.fields.size() > 65535) {
        throw new IllegalArgumentException("field limit exceeded");
    } else {
        this.cp.getClass(dotToSlash(this.className));
        this.cp.getClass("java/lang/reflect/Proxy");
        var1 = this.interfaces;
        var2 = var1.length;

        for(var3 = 0; var3 < var2; ++var3) {
            var4 = var1[var3];
            this.cp.getClass(dotToSlash(var4.getName()));
        }

        this.cp.setReadOnly();
        ByteArrayOutputStream var13 = new ByteArrayOutputStream();
        DataOutputStream var14 = new DataOutputStream(var13);

        try {
            var14.writeInt(-889275714);
            var14.writeShort(0);
            var14.writeShort(49);
            this.cp.write(var14);
            var14.writeShort(this.accessFlags);
            var14.writeShort(this.cp.getClass(dotToSlash(this.className)));
            var14.writeShort(this.cp.getClass("java/lang/reflect/Proxy"));
            var14.writeShort(this.interfaces.length);
            Class[] var17 = this.interfaces;
            int var18 = var17.length;

            for(int var19 = 0; var19 < var18; ++var19) {
                Class var22 = var17[var19];
                var14.writeShort(this.cp.getClass(dotToSlash(var22.getName())));
            }

            var14.writeShort(this.fields.size());
            var15 = this.fields.iterator();

            while(var15.hasNext()) {
                ProxyGenerator.FieldInfo var20 = (ProxyGenerator.FieldInfo)var15.next();
                var20.write(var14);
            }

            var14.writeShort(this.methods.size());
            var15 = this.methods.iterator();

            while(var15.hasNext()) {
                ProxyGenerator.MethodInfo var21 = (ProxyGenerator.MethodInfo)var15.next();
                var21.write(var14);
            }

            var14.writeShort(0);
            return var13.toByteArray();
        } catch (IOException var9) {
            throw new InternalError("unexpected I/O Exception", var9);
        }
    }
}

```

### 看看jdk生成的代理类什么样子的

上面 ProxyGenerator 中生成字节码的源码中我们看到了一个参数  sun.misc.ProxyGenerator.saveGeneratedFiles 是否将代理类写到本地，现在我们测试在运行时 修改参数

```java
System.getProperties().setProperty("sun.misc.ProxyGenerator.saveGeneratedFiles", "true");
```

然后来看看我们得到的代理类源码

![image-20200603083214248](assets/image-20200603083214248.png)

```java
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by Fernflower decompiler)
//

package com.sun.proxy;

import com.ymj.dynamic.proxy.jdk.ByeInterface;
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.lang.reflect.UndeclaredThrowableException;

public final class $Proxy0 extends Proxy implements ByeInterface {
    private static Method m1;
    private static Method m2;
    private static Method m3;
    private static Method m0;

    public $Proxy0(InvocationHandler var1) throws  {
        super(var1);
    }

    public final boolean equals(Object var1) throws  {
        try {
            return (Boolean)super.h.invoke(this, m1, new Object[]{var1});
        } catch (RuntimeException | Error var3) {
            throw var3;
        } catch (Throwable var4) {
            throw new UndeclaredThrowableException(var4);
        }
    }

    public final String toString() throws  {
        try {
            return (String)super.h.invoke(this, m2, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    public final void sayBye() throws  {
        try {
            super.h.invoke(this, m3, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    public final int hashCode() throws  {
        try {
            return (Integer)super.h.invoke(this, m0, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    static {
        try {
            m1 = Class.forName("java.lang.Object").getMethod("equals", Class.forName("java.lang.Object"));
            m2 = Class.forName("java.lang.Object").getMethod("toString");
            m3 = Class.forName("com.ymj.dynamic.proxy.jdk.ByeInterface").getMethod("sayBye");
            m0 = Class.forName("java.lang.Object").getMethod("hashCode");
        } catch (NoSuchMethodException var2) {
            throw new NoSuchMethodError(var2.getMessage());
        } catch (ClassNotFoundException var3) {
            throw new NoClassDefFoundError(var3.getMessage());
        }
    }
}
```

# cglib动态代理

它是另一种思想，通过继承的方式来达到目的。子类重写父类（被代理类）的方法，并可以对其进行增强，来实现动态代理 **被final修饰的方法，不会被子类重写**

## 使用demo

代理处理类

```java
package com.kinson.dynamic.proxy.cglib;

import net.sf.cglib.proxy.Enhancer;
import net.sf.cglib.proxy.MethodInterceptor;
import net.sf.cglib.proxy.MethodProxy;

import java.lang.reflect.Method;

public class CglibProxyHandler implements MethodInterceptor
{
    public Object CreatProxyObj(Class<?> clazz)
    {
        // 类似jdk 的 Proxy类
        Enhancer enhancer = new Enhancer();
        // 设置目标类
        enhancer.setSuperclass(clazz);
        // 设置回调函数（也就是拦截器）
        enhancer.setCallback(this);
        // 创建目标类（子类）
        return enhancer.create();
    }

    @Override
    public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable
    {
        // 前后增强
        System.out.println("Before invoke " + method.getName());
        Object object = proxy.invokeSuper(obj, args);
        System.out.println("After invoke " + method.getName());
        return object;
    }
}
```

测试用类

```java
public class Dog{
    
    final public void run(String name) {
        System.out.println("狗"+name+"----run");
    }
    
    public void eat() {
        System.out.println("狗----eat");
    }
}
```

main

```java
public class CgLibProxyDemo {
    public static void main(String[] args) {
        //在当前项目目录下生成动态代理类，我们可以反编译看一下里面到底是一些什么东西
        System.setProperty(DebuggingClassWriter.DEBUG_LOCATION_PROPERTY, System.getProperty("user.dir"));

        CglibProxyHandler cglibProxyHandler = new CglibProxyHandler();
        //这里的creat方法就是正式创建代理类
        Dog proxyDog = (Dog)cglibProxyHandler.CreatProxyObj(Dog.class);
        //调用代理类的eat方法
        proxyDog.eat(" meet ");

    }
}
```

输出结果

```
Before invoke eat
dog----eat
After invoke eat
```

## 原理

和jdk的也差不多，都是字节码重新生成，我们来看看生成后的代码吧

通过上面的demo

```java
System.setProperty(DebuggingClassWriter.DEBUG_LOCATION_PROPERTY,System.getProperty("user.dir"));
```

将cglib生成的代码写到本地自定目录，就能看到 生成了 三份代码

![image-20200603112508640](assets/image-20200603112508640.png)

其中一份是生成的代理子类，其他的是为了提升调用性能 实现的 fastClass 的子类， 为了更快的走直接调用，而不是反射调用。也是因为这个jdk 1.8以前，cglib性能是要比 jdk的Proxy快的。不过现在是 jdk的性能大于等于cglib了。

```java
public class Dog$$EnhancerByCGLIB$$7099060f extends Dog implements Factory {
    private boolean CGLIB$BOUND;
    public static Object CGLIB$FACTORY_DATA;
    private static final ThreadLocal CGLIB$THREAD_CALLBACKS;
    private static final Callback[] CGLIB$STATIC_CALLBACKS;
    private MethodInterceptor CGLIB$CALLBACK_0; // 拦截器类，也就是我们写的 MethodInterceptor的实现类
    private static Object CGLIB$CALLBACK_FILTER; 
    private static final Method CGLIB$eat$0$Method;
    private static final MethodProxy CGLIB$eat$0$Proxy;
    private static final Object[] CGLIB$emptyArgs;
    private static final Method CGLIB$equals$1$Method;
    private static final MethodProxy CGLIB$equals$1$Proxy;
    private static final Method CGLIB$toString$2$Method;
    private static final MethodProxy CGLIB$toString$2$Proxy;
    private static final Method CGLIB$hashCode$3$Method;
    private static final MethodProxy CGLIB$hashCode$3$Proxy;
    private static final Method CGLIB$clone$4$Method;
    private static final MethodProxy CGLIB$clone$4$Proxy;
	
    static {
        CGLIB$STATICHOOK1();
    }
    
    static void CGLIB$STATICHOOK1() {
        CGLIB$THREAD_CALLBACKS = new ThreadLocal();
        CGLIB$emptyArgs = new Object[0];
        Class var0 = Class.forName("com.kinson.dynamic.proxy.cglib.Dog$$EnhancerByCGLIB$$7099060f");
        Class var1;
        Method[] var10000 = ReflectUtils.findMethods(new String[]{"equals", "(Ljava/lang/Object;)Z", "toString", "()Ljava/lang/String;", "hashCode", "()I", "clone", "()Ljava/lang/Object;"}, (var1 = Class.forName("java.lang.Object")).getDeclaredMethods());
        CGLIB$equals$1$Method = var10000[0];
        CGLIB$equals$1$Proxy = MethodProxy.create(var1, var0, "(Ljava/lang/Object;)Z", "equals", "CGLIB$equals$1");
        CGLIB$toString$2$Method = var10000[1];
        CGLIB$toString$2$Proxy = MethodProxy.create(var1, var0, "()Ljava/lang/String;", "toString", "CGLIB$toString$2");
        CGLIB$hashCode$3$Method = var10000[2];
        CGLIB$hashCode$3$Proxy = MethodProxy.create(var1, var0, "()I", "hashCode", "CGLIB$hashCode$3");
        CGLIB$clone$4$Method = var10000[3];
        CGLIB$clone$4$Proxy = MethodProxy.create(var1, var0, "()Ljava/lang/Object;", "clone", "CGLIB$clone$4");
        CGLIB$eat$0$Method = ReflectUtils.findMethods(new String[]{"eat", "(Ljava/lang/String;)V"}, (var1 = Class.forName("com.kinson.dynamic.proxy.cglib.Dog")).getDeclaredMethods())[0];
        CGLIB$eat$0$Proxy = MethodProxy.create(var1, var0, "(Ljava/lang/String;)V", "eat", "CGLIB$eat$0");
    }
    
    public final void eat(String var1) {
        MethodInterceptor var10000 = this.CGLIB$CALLBACK_0;
        if (var10000 == null) {
            CGLIB$BIND_CALLBACKS(this);
            var10000 = this.CGLIB$CALLBACK_0;
        }

        if (var10000 != null) {
            var10000.intercept(this, CGLIB$eat$0$Method, new Object[]{var1}, CGLIB$eat$0$Proxy);
        } else {
            super.eat(var1);
        }
    }
    // 下面省略掉其他代码
}
```

