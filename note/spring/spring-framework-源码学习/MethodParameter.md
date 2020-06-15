# MethodParameter 对方法参数的封装

起始版本 [2.0 , )

其他它除了做方法/构造器的参数的抽象同时也可作为返回类型的抽象，在后文中可以看到

```java
private static final Annotation[] EMPTY_ANNOTATION_ARRAY = new Annotation[0];
// Executable 是 Method 或 Constructor 的公共部分的超类，属于java.lang.reflect 
private final Executable executable;
private final int parameterIndex; // 构造器或方法参数索引（标明参数的顺序）
@Nullable
private volatile Parameter parameter;// 参数
private int nestingLevel;// 签嵌套等级
@Nullable
Map<Integer, Integer> typeIndexesPerLevel;
@Nullable
private volatile Class<?> containingClass;// 所在的类
@Nullable
private volatile Class<?> parameterType;// 构造器或方法参数类型
@Nullable
private volatile Type genericParameterType;//构造器或方法参数的泛型类型
@Nullable
private volatile Annotation[] parameterAnnotations;//构造器或方法参数上的注解
@Nullable
private volatile ParameterNameDiscoverer parameterNameDiscoverer;// 参数 name 发现类
@Nullable
private volatile String parameterName;//构造器或方法参数的参数名称
@Nullable
private volatile MethodParameter nestedMethodParameter;// 嵌套参数
```

ParameterNameDiscoverer

```java
public interface ParameterNameDiscoverer {
    @Nullable
    String[] getParameterNames(Method var1);// 根据 Method 返回它的所有参数名

    @Nullable
    String[] getParameterNames(Constructor<?> var1);// 根据Constructor返回它的所有参数名
}
```

# 核心构造函数

可以看代分别处理了 Method 和 Constructor，最后都会使用他们的超类 Executable 存储，然后它的其他的方法主要是一些getXXX方法用于获取该类中字段存储的参数相关的数据。比较特别的是它有时候会作为 方法返回类型`returnType`的实现,这个可以看他的 [`getParameterType()` 方法](#getParameterType)

```java
public MethodParameter(Method method, int parameterIndex) {
    this((Method)method, parameterIndex, 1);
}

public MethodParameter(Method method, int parameterIndex, int nestingLevel) {
    this.nestingLevel = 1;
    Assert.notNull(method, "Method must not be null");
    this.executable = method;
    this.parameterIndex = validateIndex(method, parameterIndex);
    this.nestingLevel = nestingLevel;
}

public MethodParameter(Constructor<?> constructor, int parameterIndex) {
    this((Constructor)constructor, parameterIndex, 1);
}

public MethodParameter(Constructor<?> constructor, int parameterIndex, int nestingLevel) {
    this.nestingLevel = 1;
    Assert.notNull(constructor, "Constructor must not be null");
    this.executable = constructor;
    this.parameterIndex = validateIndex(constructor, parameterIndex);
    this.nestingLevel = nestingLevel;
}

public MethodParameter(MethodParameter original) {
    this.nestingLevel = 1;
    Assert.notNull(original, "Original must not be null");
    this.executable = original.executable;
    this.parameterIndex = original.parameterIndex;
    this.parameter = original.parameter;
    this.nestingLevel = original.nestingLevel;
    this.typeIndexesPerLevel = original.typeIndexesPerLevel;
    this.containingClass = original.containingClass;
    this.parameterType = original.parameterType;
    this.genericParameterType = original.genericParameterType;
    this.parameterAnnotations = original.parameterAnnotations;
    this.parameterNameDiscoverer = original.parameterNameDiscoverer;
    this.parameterName = original.parameterName;
}
```

# getParameterType

```java
public Class<?> getParameterType() {
    Class<?> paramType = this.parameterType;
    if (paramType == null) {
        if (this.parameterIndex < 0) {// 这里就是作为返回类型来使用的代码
            Method method = this.getMethod();
            paramType = method != null ? method.getReturnType() : Void.TYPE;
        } else {
            paramType = this.executable.getParameterTypes()[this.parameterIndex];
        }

        this.parameterType = paramType;
    }

    return paramType;
}
```