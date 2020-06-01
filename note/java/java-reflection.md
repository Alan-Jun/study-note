# ParameterizedType

今天在公司代码中看到了时候这样一个反射类，然后去看了一下他的作用，发现他其实是属于java泛型的范畴，从这个类的发布版本我们也能看出他他是和泛型同时发布的。

如果你定义一个类，使用了泛型，这时候通过反射就可以获取到这个类所使用的泛型参数列表，

如果没有使用泛型，也能拿到一个list的size=0的参数列表

可能这样说不太好理解，下面我们用一个例子来，直观地感受一下

```java
import java.lang.reflect.ParameterizedType;
import java.lang.reflect.Type;

public class Param<T1, T2> {
    class A {
    }

    class B extends A {
    }

    private Class<T1> entityClass;

    protected Param() {
        Type type = getClass().getGenericSuperclass();
        System.out.println("getClass()==" + getClass());
        System.out.println("type = " + type);
        Type trueType = ((ParameterizedType) type).getActualTypeArguments()[0];
        System.out.println("trueType1 = " + trueType);
        trueType = ((ParameterizedType) type).getActualTypeArguments()[1];
        System.out.println("trueType2 = " + trueType);
        this.entityClass = (Class<T1>) trueType;
		
        // 没有使用泛型的类
        B t = new B();
        type = t.getClass().getGenericSuperclass();

        System.out.println("B is A's super class :" + ((ParameterizedType) type).getActualTypeArguments().length);
    }
}
```



```java
package com.ymj.stu.reflection.parameterizedType;

public class SubParam extends Param<Integer, Number> {

    public static void main(String[] args) throws Exception {
        SubParam s = new SubParam();
    }
}
```

执行上面这个类的main()方法，我们可以看到

```
getClass()==class com.ymj.stu.reflection.parameterizedType.SubParam
type = com.ymj.stu.reflection.parameterizedType.Param<java.lang.Integer, java.lang.Number>
trueType1 = class java.lang.Integer
trueType2 = class java.lang.Number
B is A's super class :0
```

从上面的输出，我们看到了我们可以拿到对应的泛型的参数类型，拿到之后是可以转换成对应的class,然后拿到实例对象的。

