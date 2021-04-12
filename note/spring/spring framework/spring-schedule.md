# @Schedule 用法

## 参数详解

### 1. cron

该参数接收一个`cron表达式`，`cron表达式`是一个字符串，字符串以5或6个空格隔开，分开共6或7个域，每一个域代表一个含义。

#### cron表达式语法

```json
[秒] [分] [小时] [日] [月] [周] [年]
```

> 注：[年]不是必须的域，可以省略[年]，则一共6个域

| 序号 | 说明 | 必填 | 允许填写的值   | 允许的通配符  |
| ---- | ---- | ---- | -------------- | ------------- |
| 1    | 秒   | 是   | 0-59           | , - * /       |
| 2    | 分   | 是   | 0-59           | , - * /       |
| 3    | 时   | 是   | 0-23           | , - * /       |
| 4    | 日   | 是   | 1-31           | , - * ? / L W |
| 5    | 月   | 是   | 1-12 / JAN-DEC | , - * /       |
| 6    | 周   | 是   | 1-7 or SUN-SAT | , - * ? / L # |
| 7    | 年   | 否   | 1970-2099      | , - * /       |

##### 通配符说明:

- `*` 表示所有值。 例如:在分的字段上设置 *,表示每一分钟都会触发。
- `?` 表示不指定值。使用的场景为不需要关心当前设置这个字段的值。例如:要在每月的10号触发一个操作，但不关心是周几，所以需要周位置的那个字段设置为”?” 具体设置为 0 0 0 10 * ?
- `-` 表示区间。例如 在小时上设置 “10-12”,表示 10,11,12点都会触发。
- `,` 表示指定多个值，例如在周字段上设置 “MON,WED,FRI” 表示周一，周三和周五触发
- `/` 用于递增触发。如在秒上面设置”5/15” 表示从5秒开始，每增15秒触发(5,20,35,50)。 在日字段上设置’1/3’所示每月1号开始，每隔三天触发一次。
- `L` 表示最后的意思。在日字段设置上，表示当月的最后一天(依据当前月份，如果是二月还会依据是否是润年[leap]), 在周字段上表示星期六，相当于”7”或”SAT”。如果在”L”前加上数字，则表示该数据的最后一个。例如在周字段上设置”6L”这样的格式,则表示“本月最后一个星期五”
- `W` 表示离指定日期的最近那个工作日(周一至周五). 例如在日字段上置”15W”，表示离每月15号最近的那个工作日触发。如果15号正好是周六，则找最近的周五(14号)触发, 如果15号是周未，则找最近的下周一(16号)触发.如果15号正好在工作日(周一至周五)，则就在该天触发。如果指定格式为 “1W”,它则表示每月1号往后最近的工作日触发。如果1号正是周六，则将在3号下周一触发。(注，”W”前只能设置具体的数字,不允许区间”-“)。
- `#` 序号(表示每月的第几个周几)，例如在周字段上设置”6#3”表示在每月的第三个周六.注意如果指定”#5”,正好第五周没有周六，则不会触发该配置(用在母亲节和父亲节再合适不过了) ；小提示：’L’和 ‘W’可以一组合使用。如果在日字段上设置”LW”,则表示在本月的最后一个工作日触发；周字段的设置，若使用英文字母是不区分大小写的，即MON与mon相同。

##### 示例

每隔5秒执行一次：*/5 * * * * ?

每隔1分钟执行一次：0 */1 * * * ?

每天23点执行一次：0 0 23 * * ?

每天凌晨1点执行一次：0 0 1 * * ?

每月1号凌晨1点执行一次：0 0 1 1 * ?

每月最后一天23点执行一次：0 0 23 L * ?

每周星期六凌晨1点实行一次：0 0 1 ? * L

在26分、29分、33分执行一次：0 26,29,33 * * * ?

每天的0点、13点、18点、21点都执行一次：0 0 0,13,18,21 * * ?

#### cron表达式使用占位符

另外，`cron`属性接收的`cron表达式`支持占位符。eg：

配置文件：

```yaml
time:
  cron: */5 * * * * *
  interval: 5
```

每5秒执行一次：

```kotlin
    @Scheduled(cron="${time.cron}")
    void testPlaceholder1() {
        System.out.println("Execute at " + System.currentTimeMillis());
    }

    @Scheduled(cron="*/${time.interval} * * * * *")
    void testPlaceholder2() {
        System.out.println("Execute at " + System.currentTimeMillis());
    }
```

### 2. zone

时区，接收一个`java.util.TimeZone#ID`。`cron表达式`会基于该时区解析。默认是一个空字符串，即取服务器所在地的时区。比如我们一般使用的时区`Asia/Shanghai`。该字段我们一般留空。

### 3. fixedDelay

上一次执行完毕时间点之后多长时间再执行。如：

```kotlin
@Scheduled(fixedDelay = 5000) //上一次执行完毕时间点之后5秒再执行
```

### 4. fixedDelayString

与 `3. fixedDelay` 意思相同，只是使用字符串的形式。唯一不同的是支持占位符。如：

```kotlin
@Scheduled(fixedDelayString = "5000") //上一次执行完毕时间点之后5秒再执行
```

占位符的使用(配置文件中有配置：time.fixedDelay=5000)：

```css
    @Scheduled(fixedDelayString = "${time.fixedDelay}")
    void testFixedDelayString() {
        System.out.println("Execute at " + System.currentTimeMillis());
    }
```

运行结果：

 ![image-20200929104826330](/Users/ypp/Library/Application Support/typora-user-images/image-20200929104826330.png)

### 5. fixedRate

上一次开始执行时间点之后多长时间再执行。如：

```kotlin
@Scheduled(fixedRate = 5000) //上一次开始执行时间点之后5秒再执行
```

### 6. fixedRateString

与 `5. fixedRate` 意思相同，只是使用字符串的形式。唯一不同的是支持占位符。

### 7. initialDelay

第一次延迟多长时间后再执行。如：

```css
@Scheduled(initialDelay=1000, fixedRate=5000) //第一次延迟1秒后执行，之后按fixedRate的规则每5秒执行一次
```

### 8. initialDelayString

与 `7. initialDelay` 意思相同，只是使用字符串的形式。唯一不同的是支持占位符。

That's all ! Thanks for reading.

# 附：

截至`spring-context-4.3.14.RELEASE`的源码

```dart
/**
 * An annotation that marks a method to be scheduled. Exactly one of
 * the {@link #cron()}, {@link #fixedDelay()}, or {@link #fixedRate()}
 * attributes must be specified.
 *
 * <p>The annotated method must expect no arguments. It will typically have
 * a {@code void} return type; if not, the returned value will be ignored
 * when called through the scheduler.
 *
 * <p>Processing of {@code @Scheduled} annotations is performed by
 * registering a {@link ScheduledAnnotationBeanPostProcessor}. This can be
 * done manually or, more conveniently, through the {@code <task:annotation-driven/>}
 * element or @{@link EnableScheduling} annotation.
 *
 * <p>This annotation may be used as a <em>meta-annotation</em> to create custom
 * <em>composed annotations</em> with attribute overrides.
 *
 * @author Mark Fisher
 * @author Dave Syer
 * @author Chris Beams
 * @since 3.0
 * @see EnableScheduling
 * @see ScheduledAnnotationBeanPostProcessor
 * @see Schedules
 */
@Target({ElementType.METHOD, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Repeatable(Schedules.class)
public @interface Scheduled {

    /**
     * A cron-like expression, extending the usual UN*X definition to include
     * triggers on the second as well as minute, hour, day of month, month
     * and day of week.  e.g. {@code "0 * * * * MON-FRI"} means once per minute on
     * weekdays (at the top of the minute - the 0th second).
     * @return an expression that can be parsed to a cron schedule
     * @see org.springframework.scheduling.support.CronSequenceGenerator
     */
    String cron() default "";

    /**
     * A time zone for which the cron expression will be resolved. By default, this
     * attribute is the empty String (i.e. the server's local time zone will be used).
     * @return a zone id accepted by {@link java.util.TimeZone#getTimeZone(String)},
     * or an empty String to indicate the server's default time zone
     * @since 4.0
     * @see org.springframework.scheduling.support.CronTrigger#CronTrigger(String, java.util.TimeZone)
     * @see java.util.TimeZone
     */
    String zone() default "";

    /**
     * Execute the annotated method with a fixed period in milliseconds between the
     * end of the last invocation and the start of the next.
     * @return the delay in milliseconds
     */
    long fixedDelay() default -1;

    /**
     * Execute the annotated method with a fixed period in milliseconds between the
     * end of the last invocation and the start of the next.
     * @return the delay in milliseconds as a String value, e.g. a placeholder
     * @since 3.2.2
     */
    String fixedDelayString() default "";

    /**
     * Execute the annotated method with a fixed period in milliseconds between
     * invocations.
     * @return the period in milliseconds
     */
    long fixedRate() default -1;

    /**
     * Execute the annotated method with a fixed period in milliseconds between
     * invocations.
     * @return the period in milliseconds as a String value, e.g. a placeholder
     * @since 3.2.2
     */
    String fixedRateString() default "";

    /**
     * Number of milliseconds to delay before the first execution of a
     * {@link #fixedRate()} or {@link #fixedDelay()} task.
     * @return the initial delay in milliseconds
     * @since 3.2
     */
    long initialDelay() default -1;
    String initialDelayString() default "";

}
```

> 作者：sprinkle_liz
> 链接：https://www.jianshu.com/p/1defb0f22ed1
> 来源：简书
> 著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

## 原理