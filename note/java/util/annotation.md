# java.lang.annotation 下的java基础注解

下面我们将会看到我们在自定义注解的时候会用到的一些从1.5开始就有了的基础注解，以及两个1.8新增的注解 `@Native` 和`@Repeatable`

## @Target：

**作用：用于描述注解的使用范围（即：被描述的注解可以用在什么地方）**

```java
/*
 * @since 1.5
 * @jls 9.6.4.1 @Target
 * @jls 9.7.4 Where Annotations May Appear
 */
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Target {
    /**
     * Returns an array of the kinds of elements an annotation type
     * can be applied to.
     * @return an array of the kinds of elements an annotation type
     * can be applied to
     */
    ElementType[] value();
}
```



**ElementType 取值有：**

| **field**                            | **description**                                              |
| ------------------------------------ | ------------------------------------------------------------ |
| **`TYPE`**                           | 可用于描述类、接口(包括注解类型) 或enum声明 <br />**Class, interface (including annotation type), or enum declaration** |
| **`FIELD`**                          | 可用于描述字段  <br />**Field declaration (includes enum constants)** |
| **`METHOD`**                         | 可用于描述方法<br />**Method declaration**                   |
| **`PARAMETER`**                      | 可用于描述形参<br />**Formal parameter declaration**         |
| **`CONSTRUCTOR`**                    | 可用于描述构造方法<br />**Constructor declaration**          |
| **`LOCAL_VARIABLE`**                 | 可用于描述局部变量<br />**Local variable declaration**       |
| **`ANNOTATION_TYPE`**                | 可用于描述注解<br />**Annotation type declaration**          |
| **`PACKAGE`**                        | 可用于package<br />**Package declaration**                   |
| **`TYPE_PARAMETER`** <br />since 1.8 | 对枚举方法的类型就行修饰( 个人不是很确定，没使用过 )<br />**Type parameter declaration** |
| **`TYPE_USE`**<br />since 1.8        | 表示该注解能使用在使用类型的任意语句中。<br/>**Use of a type** |
| **``**                               |                                                              |

## @Retention

**作用：表示需要在什么级别保存该注释信息，用于描述注解的生命周期（即：被描述的注解在什么范围内有效）**

```java
/*
 * @author  Joshua Bloch
 * @since 1.5
 * @jls 9.6.3.2 @Retention
 */
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Retention {
    /**
     * Returns the retention policy.
     * @return the retention policy
     */
    RetentionPolicy value();
}
```

**RetentionPolicy 取值有：**

* SOURCE:在源文件中有效（即源文件保留）
* CLASS:在class文件中有效（即class保留）
* RUNTIME:在运行时有效（即运行时保留）

## @Inherited

@Inherited 元注解是一个标记注解，表明子类可以继承父类中的该注解

```java
/**
 * @author  Joshua Bloch
 * @since 1.5
 * @jls 9.6.3.3 @Inherited
 */
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Inherited {
}
```

## @Documented:

**@**Documented用于描述其它类型的annotation应该被作为被标注的程序成员的公共API，因此可以被例如javadoc此类的工具文档化。Documented是一个标记注解，没有成员。

```java
/**
 * @author  Joshua Bloch
 * @since 1.5
 */
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Documented {
}
```

## @Repeatable

```java
/**
 * The annotation type {@code java.lang.annotation.Repeatable} is
 * used to indicate that the annotation type whose declaration it
 * (meta-)annotates is <em>repeatable</em>. The value of
 * {@code @Repeatable} indicates the <em>containing annotation
 * type</em> for the repeatable annotation type.
 *
 * @since 1.8
 * @jls 9.6 Annotation Types
 * @jls 9.7 Annotations
 */
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Repeatable {
    /**
     * Indicates the <em>containing annotation type</em> for the
     * repeatable annotation type.
     * @return the containing annotation type
     */
    Class<? extends Annotation> value();
}
```

## @Native

```java
/**
 * Indicates that a field defining a constant value may be referenced
 * from native code.
 *
 * The annotation may be used as a hint by tools that generate native
 * header files to determine whether a header file is required, and
 * if so, what declarations it should contain.
 *
 * @since 1.8
 */
@Documented
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.SOURCE)
public @interface Native {
}
```

# common annotation

Common annotations原本是Java EE 5.0(JSR 244)规范的一部分，现在SUN把它的一部分放到了Java SE 6.0中.

目前完整的 `Common Annotation`协议的注解都在下面的依赖中，javax.annotation 包下面也有一部分，但是完整的都在下面这个依赖包中

```xml
<dependency>
  <groupId>jakarta.annotation</groupId>
  <artifactId>jakarta.annotation-api</artifactId>
  <version>1.3.5</version><!--最新版本去maven仓库中找-->
  <scope>compile</scope>
</dependency>
```

maven 依赖 这个包中的注解都来自 jdk 1.6 引入 `Common Annotation` 协议了，具体来自哪一个协议，这个可以去看具体的注解定义的源代码，注释信息描述得很清楚，有来自 `Common Annotations 1.0` 也有`Common Annotations 1.1`,`Common Annotations 1.2`或则后续的版本

## common annotation 1.0

主要有10个注解，其中五个在 javax.annotation 包下，还有五个在 javax.annotation.security 包下

**javax.annotation**

| Annotation     | @Target                                                      | @Retention | @Inherited | Description                                                  |
| -------------- | ------------------------------------------------------------ | ---------- | ---------- | ------------------------------------------------------------ |
| @Generated     | `ANNOTATION_TYPE`, `CONSTRUCTOR`, <br/>`FIELD`, `LOCAL_VARIABLE`, `METHOD`, <br/>`PACKAGE`,` PARAMETER`, `TYPE` | SOURCE     | non        | 用于标注生成的源代码                                         |
| @Resource      | `TYPE`, `METHOD`,` FIELD`                                    | Runtime    | non        | 用于标注所依赖的资源,<br/>容器据此注入外部资源依赖，<br/>有基于字段的注入和基于setter方法的注入两种方式 |
| @Resources     | `TYPE`                                                       | RUNTIME    | non        | 同时标注多个外部依赖，容器会把所有这些外部依赖注入           |
| @PostConstruct | `METHOD`                                                     | Runtime    | non        | 标注当容器注入所有依赖之后将运行的方法，用来进行依赖注入后的初始化工作，只有一个方法可以标注为@PostConstruct |
| @PreDestroy    | `METHOD`                                                     | Runtime    | non        | 当对象实例将要被从容器当中删掉之前，要执行的回调方法要标注为@PreDestroy |

**javax.annotation.security**

| Annotation    | @Target      | @Retention | @Inherited | Description |
| ------------- | ------------ | ---------- | ---------- | ----------- |
| @RunAs        | `TYPE`       | RUNTIME    | non        |             |
| @PermitAll    | TYPE，METHOD | RUNTIME    | non        |             |
| @DenyAll      | TYPE，METHOD | RUNTIME    | non        |             |
| @DeclareRoles | TYPE         | RUNTIME    | non        |             |
| @RolesAllowed | TYPE，METHOD | RUNTIME    | non        |             |

## common annotation 1.1 

| Annotation             | @Target | @Retention | @Repeatable                                   | Description |
| ---------------------- | ------- | ---------- | --------------------------------------------- | ----------- |
| @DataSourceDefinition  | TYPE    | RUNTIME    | @Repeatable(<br/>DataSourceDefinitions.class) |             |
| @DataSourceDefinitions | TYPE    | RUNTIME    |                                               |             |
| @ManagedBean           | TYPE    | RUNTIME    |                                               |             |
|                        |         |            |                                               |             |
|                        |         |            |                                               |             |

## common annotation 1.2

| Annotation | @Target        | @Retention | @Repeatable | Description |
| ---------- | -------------- | ---------- | ----------- | ----------- |
| @Priority  | TYPE,PARAMETER | RUNTIME    |             |             |
|            |                |            |             |             |
|            |                |            |             |             |
|            |                |            |             |             |
|            |                |            |             |             |

# 空注释

- `@NonNull`：空值是非法值
- `@Nullable`：允许使用空值，且必须是预期的值
- `@NonNullByDefault`：缺少空注释的方法特征符的类型被视为非空。

在以下位置支持注释 `@NonNull` 和 `@Nullable`：

- 方法参数
- 方法返回（在此处按句法使用方法注释）
- 局部变量
- 字段
- 在 Java 8 中，可使用[空类型注释](https://www.ibm.com/support/knowledgecenter/zh/SS8PJ7_9.6.1/org.eclipse.jdt.doc.user/tasks/task-using_null_type_annotations.htm?view=kc)来注释更多位置

`@NonNullByDefault` 受以下内容支持：

- 方法 - 影响此方法的特征符中所有的类型
- 类型（类、接口和枚举）- 影响类型主体中的所有方法
- 程序包（通过 `package-info.java` 文件）- 影响程序包中的所有类型

注意：即使这些注释的实际限定名为[可配置](https://www.ibm.com/support/knowledgecenter/zh/SS8PJ7_9.6.1/org.eclipse.jdt.doc.user/reference/preferences/java/compiler/ref-preferences-errors-warnings.htm?view=kc#null_annotation_names)，但在缺省情况下，仍会（从程序包 `org.eclipse.jdt.annotation`）使用上面给出的限定名。使用第三方空注释时，请确保它们是使用至少一个 `@Target` 元注释正确定义的，（因为）否则编译器无法区分声明注释 (Java 5) 与类型注释 (Java 8)。