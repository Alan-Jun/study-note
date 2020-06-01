# CAS 介绍

`CAS`compare and swap的缩写，也就是比较再替换的意思，`CAS` 操作要求三个参数：内存地址 V，旧的预期值 O，要修改的新值 N，在更新的时候，需要V这个内存地址的值和当前传递过来的参数O的值相同，才能执行修改操作，**CAS指令是一个CPU层级的原子性操作指令。 在 Intel 处理器中， 其汇编指令为 cmpxchg。**。

## CAS可以做什么

有了这个操作我们可以实现无锁并发，用来解决并发编程中的部分并发安全问题,比如java中java.util.concurrent包下面得那些 原子操作类 例如 `AtomicInteger` 

> java中原子类的详细介绍请看 [JUC](JUC.md)的原子类的介绍

# CAS 操作的 ABA 问题

* **AbA问题**

  要了解什么是ABA问题，首先我们来通俗的看一下这个例子，一家火锅店为了生意推出了一个特别活动，凡是在五一期间的老用户凡是卡里余额小于20的，赠送10元，但是这种活动没人只可享受一次。然后火锅店的后台程序员小王开始工作了，很简单就用cas技术，先去用户卡里的余额，然后包装成AtomicInteger，写一个判断，开启10个线程，然后判断小于20的，一律加20，然后就很开心的交差了。可是过了一段时间，发现账面亏损的厉害，老板起先的预支是2000块，因为店里的会员总共也就100多个，就算每人都符合条件，最多也就2000啊，怎么预支了这么多。小王一下就懵逼了，赶紧debug，tail -f一下日志，这不看不知道，一看吓一跳，有个客户被充值了10次!

  阐述：

  假设有个线程A去判断账户里的钱此时是15，满足条件，直接+20，这时候卡里余额是35.但是此时不巧，正好在连锁店里，这个客人正在消费，又消费了20，此时卡里余额又为15，线程B去执行扫描账户的时候，发现它又小于20，又用过cas给它加了20，这样的话就相当于加了两次，这样循环往复肯定把老板的钱就坑没了！

  本质：

  **ABA问题的根本在于cas在修改变量的时候，无法记录变量的状态（也就是值虽然相同，但是我们不知道这个值什么条件得来的），比如修改的次数，是否修改过这个变量。这样就很容易在一个线程将A修改成B时，另一个线程又会把B修改成A,造成cas多次执行的问题。**

* **如果自旋时间较长的性能开销问题**
* **只能保证一个共享变量的原子操作**
  
  * 当然也有解决方法，比如 AtomicReference 类

# CAS java 中的函数介绍

CAS(compare and swap)，比较之后交换。这事一个现在处理器都提供的一个命令，在java中由sun.misc.Unsafe 这个工具类，提供一系列的实现（这个类在外部不能直接调用）：

* `final native boolean compareAndSwapObject(Object paramObject, long valueOffset, Object expect, Object update)`
* `final native boolean compareAndSwapObject(Object paramObject, long valueOffset, Object expect, Object update)`
* `final native boolean compareAndSwapObject(Object paramObject, long valueOffset, Object expect, Object update)`

参数都是一样的。这里我们解释一下参数的意义：

* `paramObject`: 需要操作的对象，一般都是当前对象（因为我们使用的都是jdk提供的封装类）。
* `valueOffset`：操作对象中封装的属性的内存偏移量，c++本地函数通过这个偏移量获取当前对象内存地址中具体的这个属性的内存地址
* `expect`：旧的预期的值，如果是这个值，才能修改成功
* `update`：即将更新到的值

## 8.2 CAS 内部原理

* 使用 上面 CAS 方法的程序，如果在多处理器（多cpu）环境运行，就会为其加上 lock 前缀，带有这个前缀的该cas指令在现代的处理器中会使用缓存锁定并且搭配缓存一致性协议来保证这个变量的执行的原子性。
* 该指令的前后的读/写指令会被禁止重排序。
* 执行之后会将该写缓冲区的所有数据刷新到主内存。

**总结**：可以说该指令 同时实现了volatile 读和写的内存语义。