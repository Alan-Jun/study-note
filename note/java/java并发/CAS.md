# CAS 介绍

`CAS`compare and swap的缩写，也就是比较再替换的意思，`CAS` 操作要求三个参数：内存地址 V，旧的预期值 O，要修改的新值 N，在更新的时候，需要V这个内存地址的值和当前传递过来的参数O的值相同，才能执行修改操作，**CAS指令是一个CPU层级的原子性操作指令。 在 Intel 处理器中， 其汇编指令为 cmpxchg。**

## CAS可以做什么

有了这个操作我们可以实现无锁并发，用来解决并发编程中的部分并发安全问题,比如java中java.util.concurrent包下面得那些 原子操作类 例如 `AtomicInteger` 

CAS的思想也被我们广泛使用，以及扩展使用，不在只是存在于我们的汇编指令层面，很多涉及到一些并发的场景我们也会做类似的设计，比如不在只是局限于等于某一个值才去修改，可以变成小于 O , 甚至说是某一个数据小于某一个版本怎么做，大于某一个版本怎么处理；比如我们的 MVCC（多版本并发控制） 就是某些数据大于等于某一个版本的可见，小于的不可见，当然我不是很清楚现有CAS还是先有MVCC,不过设计思路上我任务他们很相似

> java中原子类的详细介绍请看 [JUC](JUC.md)的原子类的介绍

## CAS 操作存在的问题

* **AbA问题** ： ABA的问题指的是在CAS更新的过程中，当读取到的值是A，然后准备赋值的时候仍然是A，但是实际上有可能A的值被改成了B，然后又被改回了A，这个CAS更新的漏洞就叫做ABA。只是ABA的问题大部分场景下都不影响并发的最终效果。

  Java中有AtomicStampedReference来解决这个问题，他加入了预期标志和更新后标志两个字段，更新时不光检查值，还要检查当前的标志是否等于预期标志，全部相等的话才会更新。

* **自旋循环时间长开销大**：如果使用CAS的时候采用了自旋操作，那么可能会造成性能开销问题：因为每次检查不符合预期的话自旋逻辑都会不断重试，那这个等待的时间可能会很长，CPU开销会很大

* **只能保证一个共享变量的原子操作**

## CAS java 中的函数介绍

CAS(compare and swap)，比较之后交换。这事一个现在处理器都提供的一个命令，在java中由sun.misc.Unsafe 这个工具类，提供一系列的实现（这个类在外部不能直接调用）：

* `final native boolean compareAndSwapObject(Object paramObject, long valueOffset, Object expect, Object update)`
* `final native boolean compareAndSwapObject(Object paramObject, long valueOffset, Object expect, Object update)`
* `final native boolean compareAndSwapObject(Object paramObject, long valueOffset, Object expect, Object update)`

参数都是一样的。这里我们解释一下参数的意义：

* `paramObject`: 需要操作的对象，一般都是当前对象（因为我们使用的都是jdk提供的封装类）。
* `valueOffset`：操作对象中封装的属性的内存偏移量，c++本地函数通过这个偏移量获取当前对象内存地址中具体的这个属性的内存地址
* `expect`：旧的预期的值，如果是这个值，才能修改成功
* `update`：即将更新到的值

## CAS 内部原理

* 使用上面 CAS 方法的程序，如果在多处理器（多cpu）环境运行，就会为其加上 lock 前缀，带有这个前缀的该cas指令在现代的处理器中会使用缓存锁定并且搭配缓存一致性协议来保证这个变量的执行的原子性。
* 该指令的前后的读/写指令会被禁止重排序。
* 执行之后会将该写缓冲区的所有数据刷新到主内存。

