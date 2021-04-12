

# jvm启动模式配置

-client

 设置jvm使用client模式，特点是启动速度比较快，但运行时性能和内存管理效率不高，通常用于客户端应用程序或者PC应用开发和调试。 

-server

 设置jvm使server模式，特点是启动速度比较慢，但运行时性能和内存管理效率很高，适用于生产环境。在具有64位能力的jdk环境下将默认启用该模式，而忽略-client参数。 

# jdk1.7 方法区内存配置参数

-XX:MaxPermSize=128m  

-XX:PermSize=128m  

# jdk1.8 metaSpace配置参数

| 参数                      | 作用                                                         |
| ------------------------- | ------------------------------------------------------------ |
| -XX:MetaspaceSize         | **配置初始化的Metaspace大小，控制Metaspace发生GC的阈值**。GC后，动态增加或者降低MetaspaceSize，默认情况下，这个值大小根据不同的平台在12M到20M之间浮动 |
| -XX:MaxMetaspaceSize      | **限制Metaspace增长上限，防止因为某些情况导致Metaspace无限使用本地内存，影响到其他程序**，默认为4096M |
| -XX:MinMetaspaceFreeRatio | **当进行过Metaspace GC之后，会计算当前Metaspace的空闲空间比，如果空闲比小于这个参数，那么虚拟机   *增长*   Metaspace的大小**，默认为40，即40% |
| -XX:MaxMetaspaceFreeRatio | **当进行过Metaspace GC之后，会计算当前Metaspace的空闲空间比，如果空闲比大于这个参数，那么虚拟机会   *释放*    部分Metaspace空间**，默认为70，即70% |
| -XX:MetaspaceExpanison    | **Metaspace增长时的最大幅度，默认值为5M**                    |
| -XX:MinMetaspaceExpanison | **Metaspace增长时的最小幅度**，默认为330KB                   |



# 堆内存配置参数

## -Xms

指定jvm堆的初始大小，默认为物理内存的1/64，最小为1M；可以指定单位，比如k、m，若不指定，则默认为字节。 

```
-Xms128m
```

## -Xmx

指定jvm堆的最大值，默认为物理内存的1/4或者1G，最小为2M；单位与-Xms一致。 

```
-Xmx512m
```

# 线程栈大小配置参数

-Xss

 设置单个线程栈的大小，一般默认为512k。

```
-Xss512K
```

#  codecache配置参数

```
-XX:InitialCodeCacheSize=100m  

-XX:ReservedCodeCacheSize=240m
```

# 输出cpu配置文件数据

```
 -Xprof 
```

# 在JVM启动后，在命令行中可以输出所有XX参数和值 

jdk1.6以后  

```
-XX:+PrintFlagsFinal  -XX:+PrintFlagsInitial
```

# JVM行为参数

## -XX:-DisableExplicitGC

  禁止调用System.gc()；但jvm的gc仍然有效

## -XX:+MaxFDLimit

最大化文件描述符的数量限制 

## -XX:+ScavengeBeforeFullGC

新生代GC优先于Full GC执行 

## -XX:+UseGCOverheadLimit 

在抛出OOM之前限制jvm耗费在GC上的时间比例

## -XX:-UseConcMarkSweepGC 

对老年代采用CMS收集器

## -XX:+UseG1GC

使用G1做了垃圾收集器

## -XX:-UseParallelGC  

启用并行GC 

## -XX:-UseParallelOldGC

对老年代启用并行收集器，当-XX:-UseParallelGC启用时该项自动启用 

## -XX:-UseSerialGC  

启用 Serial 收集器

## -XX:+UseThreadPriorities    

启用本地线程优先级 

## -XX:+ParallelRefProcEnabled

默认为false，指定这个参数的话就相当于激活这个功能，并行的处理Reference对象，如WeakReference，除非在GC log里出现Reference处理时间较长的日志，否则效果不会很明显。

# GC日志参数

GC过程可以通过GC日志来提供优化依据。

## -XX:+PrintGCDetails

启用gc日志打印功能

## -Xloggc:/path/to/gc.log：

指定gc日志位置，例如

```
-Xloggc:/data/logs/gc.log
```

## -XX:+PrintHeapAtGC

打印GC前后的详细堆栈信息

## -XX:+PrintGCDateStamps

打印可读的日期而不是时间戳

## -XX:+PrintGCApplicationStoppedTime

打印所有引起JVM停顿时间，如果真的发现了一些不知什么的停顿，再临时加上`-XX:+PrintSafepointStatistics -XX: PrintSafepointStatisticsCount=1`找原因。

## -XX:+PrintGCApplicationConcurrentTime

打印JVM在两次停顿之间正常运行时间，与`-XX:+PrintGCApplicationStoppedTime`一起使用效果更佳。

## -XX:+PrintTenuringDistribution

查看每次minor GC(新生代gc)后新的存活周期的阈值

## XX:+UseGCLogFileRotation 

与 -XX:NumberOfGCLogFiles=10 与 -XX:GCLogFileSize=10M：GC日志在重启之后会清空，但是如果一个应用长时间不重启，那GC日志会增加，所以添加这3个参数，是GC日志滚动写入文件，但是如果重启，可能名字会出现混乱。

## -XX:PrintFLSStatistics=1

打印每次GC前后内存碎片的统计信息

# G1参数

## -XX:G1HeapRegionSize

一个Region的大小可以通过参数-XX:G1HeapRegionSize设定，取值范围从1M到32M，且是2的指数。如果不设定，那么G1会根据Heap大小自动决定

## -XX:MaxGCPauseMillis

设置G1收集的目标时间，默认是200ms,G1会去尽量达到这个目标收集时间，但是不能保证每次暂停一定在这个要求之内。根据测试发现，如果我们将这个值设定成50毫秒或者更低的话，JVM为了达到这个要求会将年轻代内存空间设定的非常小，从而导致youngGC的频率大大增高。

## -XX:InitiatingHeapOccupancyPercent

设置触发标记周期的 Java 堆占用率阈值。默认占用率是整个 Java 堆的 45%。就是说当使用内存占到堆总大小的45%的时候，G1将开始**并发标记阶段。**为混合GC做准备，这个数值在测试的时候我想让混合GC晚一些处理所以设定成了70%，经过观察发现如果这个数值设定过大会导致JVM无法启动并发标记，直接进行FullGC处理。G1的FullGC是单线程，一个22G的对GC完成需要8S的时间，所以这个值在调优的时候写的45%



> https://blog.csdn.net/liuxinghao/article/details/73963399