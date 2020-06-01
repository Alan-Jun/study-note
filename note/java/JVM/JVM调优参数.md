

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

-Xms

指定jvm堆的初始大小，默认为物理内存的1/64，最小为1M；可以指定单位，比如k、m，若不指定，则默认为字节。 

```
-Xms128m
```

-Xmx

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

```
-XX:-DisableExplicitGC  禁止调用System.gc()；但jvm的gc仍然有效 
-XX:+MaxFDLimit 最大化文件描述符的数量限制 
-XX:+ScavengeBeforeFullGC   新生代GC优先于Full GC执行 
-XX:+UseGCOverheadLimit 在抛出OOM之前限制jvm耗费在GC上的时间比例 
-XX:-UseConcMarkSweepGC 对老生代采用并发标记交换算法进行GC 
-XX:-UseParallelGC  启用并行GC 
-XX:-UseParallelOldGC   对Full GC启用并行，当-XX:-UseParallelGC启用时该项自动启用 
-XX:-UseSerialGC    启用串行GC 
-XX:+UseThreadPriorities    启用本地线程优先级 
```

