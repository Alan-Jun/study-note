# 申明

这部分内容很多的注解都没有归类，它是属于jdk的还是属于哪一个框架定义的。

# @SuppressWarnings

**用于抑制编译器产生警告信息。**

```java
@Target({TYPE, FIELD, METHOD, PARAMETER, CONSTRUCTOR, LOCAL_VARIABLE})
@Retention(RetentionPolicy.SOURCE)
public @interface SuppressWarnings {
    /**
     * The set of warnings that are to be suppressed by the compiler in the
     * annotated element.  Duplicate names are permitted.  The second and
     * successive occurrences of a name are ignored.  The presence of
     * unrecognized warning names is <i>not</i> an error: Compilers must
     * ignore any warning names they do not recognize.  They are, however,
     * free to emit a warning if an annotation contains an unrecognized
     * warning name.
     *
     * <p> The string {@code "unchecked"} is used to suppress
     * unchecked warnings. Compiler vendors should document the
     * additional warning names they support in conjunction with this
     * annotation type. They are encouraged to cooperate to ensure
     * that the same names work across multiple compilers.
     * @return the set of warnings to be suppressed
     */
    String[] value();
}
```

**通过源码我们知道这个注解可以用于，类，字段，方法，方法参数，构造函数，方法局部变量**

## 参数类型

| **关键字**               | **用途**                                                     |
| ------------------------ | ------------------------------------------------------------ |
| all                      | to suppress all warnings                                     |
| boxing                   | to suppress warnings relative to boxing/unboxing operations  |
| cast                     | to suppress warnings relative to cast operations             |
| dep-ann                  | to suppress warnings relative to deprecated annotation       |
| deprecation              | to suppress warnings relative to deprecation                 |
| fallthrough              | to suppress warnings relative to missing breaks in switch statements |
| finally                  | to suppress warnings relative to finally block that don’t return |
| hiding                   | to suppress warnings relative to locals that hide variable   |
| incomplete-switch        | to suppress warnings relative to missing entries in a switch statement (enum case) |
| nls                      | to suppress warnings relative to non-nls string literals     |
| null                     | to suppress warnings relative to null analysis               |
| rawtypes                 | to suppress warnings relative to un-specific types when using generics on class params |
| restriction              | to suppress warnings relative to usage of discouraged or forbidden references |
| serial                   | to suppress warnings relative to missing serialVersionUID field for a serializable class |
| static-access            | o suppress warnings relative to incorrect static access      |
| synthetic-access         | to suppress warnings relative to unoptimized access from inner classes |
| unchecked                | to suppress warnings relative to unchecked operations        |
| unqualified-field-access | to suppress warnings relative to field access unqualified    |
| unused                   | to suppress warnings relative to unused code                 |

## 示例

* 抑制单类型的warning

  ```java
  @SuppressWarnings("unchecked")
  public void addItems(String item){
    @SuppressWarnings("rawtypes")
     List items = new ArrayList();
     items.add(item);
  }
  ```

* 抑制多类型的警告

  ```java
  @SuppressWarnings(value={"unchecked", "rawtypes"})
  public void addItems(String item){
     List items = new ArrayList();
     items.add(item);
  }
  ```

* 抑制所有类型的警告

  ```java
  @SuppressWarnings("all")
  public void addItems(String item){
     List items = new ArrayList();
     items.add(item);
  }
  ```

