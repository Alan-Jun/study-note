* 1.7 开始 静态常量池 就移动到了 java heap
* 1.8 移除了PermGen Space，新增了，Metaspace
  * Metaspace 是一个 可伸缩

**移除 PermGen Space 作为 类元数据存放区域的原因**

* 为了和JRockit进行融合而做的努力，Rockit用户并不需要配置持久代（因为JRockit就没有持久代）

* PermGen Space 大小收到-XX：PermSize和-XX：MaxPermSize参数限制，这个又是受到 JVM 的内存限制的，那么 PermGen Space OOM 的可能性较大

**Metaspace 的优点**

* Metaspace 的内存并不是在JVM中分配的，使用的是本地内存（直接内存），大小是可动态的变化的，大小受到32/64位系统内存可用大小的限制，当然也可以使用JVM提供的对应参数来配置 [参考JVM参数配置](JVM调优参数.md)

  



