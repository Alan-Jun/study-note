# 强引用（StrongReference）

​    强引用是使用最普遍的引用。如果一个对象具有强引用，那垃圾回收器绝不会回收它。如下：

```java
Object o=new Object();   //  强引用
```

​    当内存空间不足，Java虚拟机宁愿抛出OutOfMemoryError错误，使程序异常终止，也不会靠随意回收具有强引用的对象来解决内存不足的问题。如果不使用时，要通过如下方式来弱化引用，如下：

```java
o=null;     // 帮助垃圾收集器回收此对象
```

​    显式地设置o为null，或超出对象的生命周期范围，则gc认为该对象不存在引用，这时就可以回收这个对象。具体什么时候收集这要取决于gc的算法。

举例：

```java
public void test(){
    Object o=new Object();
    // 省略其他操作
}
```

​    在一个方法的内部有一个强引用，这个引用保存在栈中，而真正的引用内容（Object）保存在堆中。当这个方法运行完成后就会退出方法栈，则引用内容的引用不存在，这个Object会被回收。

​    但是如果这个o是全局的变量时，就需要在不用这个对象时赋值为null，因为强引用不会被垃圾回收。

​    强引用在实际中有非常重要的用处，举个ArrayList的实现源代码：

```java
private transient Object[] elementData;
public void clear() {
        modCount++;
        // Let gc do its work
        for (int i = 0; i < size; i++)
            elementData[i] = null;
        size = 0;
}
```

在ArrayList类中定义了一个私有的变量elementData数组，在调用方法清空数组时可以看到为每个数组内容赋值为null。不同于elementData=null，强引用仍然存在，避免在后续调用 add()等方法添加元素时进行重新的内存分配。使用如clear()方法中释放内存的方法对数组中存放的引用类型特别适用，这样就可以及时释放内存。 

# 软引用（SoftReference）

   如果一个对象只具有软引用，则内存空间足够，垃圾回收器就不会回收它；如果内存空间不足了，就会回收这些对象的内存。只要垃圾回收器没有回收它，该对象就可以被程序使用。软引用可用来实现内存敏感的高速缓存。   

```java
String str=new String("abc");                                     // 强引用
SoftReference<String> softRef=new SoftReference<String>(str);     // 软引用
```

当内存不足时，等价于：  

```java
If(JVM.内存不足()) {
   str = null;  // 转换为软引用
   System.gc(); // 垃圾回收器进行回收
}
```

 软引用在实际中有重要的应用，例如浏览器的后退按钮。按后退时，这个后退时显示的网页内容是重新进行请求还是从缓存中取出呢？这就要看具体的实现策略了。

（1）如果一个网页在浏览结束时就进行内容的回收，则按后退查看前面浏览过的页面时，需要重新构建

（2）如果将浏览过的网页存储到内存中会造成内存的大量浪费，甚至会造成内存溢出

这时候就可以使用软引用

```java
Browser prev = new Browser();               // 获取页面进行浏览
SoftReference sr = new SoftReference(prev); // 浏览完毕后置为软引用        
if(sr.get()!=null){ 
    rev = (Browser) sr.get();           // 还没有被回收器回收，直接获取
}else{
    prev = new Browser();               // 由于内存吃紧，所以对软引用的对象回收了
    sr = new SoftReference(prev);       // 重新构建
}
```

   这样就很好的解决了实际的问题。

   软引用可以和一个引用队列（ReferenceQueue）联合使用，如果软引用所引用的对象被垃圾回收器回收，Java虚拟机就会把这个软引用加入到与之关联的引用队列中。

# 弱引用（WeakReference）

## 什么是弱引用？

弱引用与软引用的区别在于：只具有弱引用的对象拥有更短暂的生命周期。在垃圾回收器线程扫描它所管辖的内存区域的过程中，一旦发现了**只**具有弱引用的对象，不管当前内存空间足够与否，都会回收它的内存。不过，由于垃圾回收器是一个优先级很低的线程，因此不一定会很快发现那些只具有弱引用的对象。 

## 为什么使用弱引用？

考虑下面的场景：现在有一个Product类代表一种产品，这个类被设计为不可扩展的，而此时我们想要为每个产品增加一个编号。一种解决方案是使用HashMap<Product, Integer>。于是问题来了，如果我们已经不再需要一个Product对象存在于内存中（比如已经卖出了这件产品），假设指向它的引用为productA，我们这时会给productA赋值为null，然而这时productA过去指向的Product对象并不会被回收，因为它显然还被HashMap引用着。所以这种情况下，我们想要真正的回收一个Product对象，仅仅把它的强引用赋值为null是不够的，还要把相应的条目从HashMap中移除。显然“从HashMap中移除不再需要的条目”这个工作我们不想自己完成，我们希望告诉垃圾收集器：在只有HashMap中的key在引用着Product对象的情况下，就可以回收相应Product对象了。显然，根据前面弱引用的定义，使用弱引用能帮助我们达成这个目的。我们只需要用一个指向Product对象的弱引用对象来作为HashMap中的key就可以了。

## 如何使用弱引用？

拿上面介绍的场景举例，我们使用一个指向Product对象的弱引用对象来作为HashMap的key，只需这样定义这个弱引用对象：

```java
Product productA = new Product(...);
WeakReference<Product> weakProductA = new WeakReference<>(productA);
```

现在，若引用对象weakProductA就指向了Product对象productA。那么我们怎么通过weakProduct获取它所指向的Product对象productA呢？很简单，只需要下面这句代码：

```java
Product product = weakProductA.get();
```

实际上，对于这种情况，Java类库为我们提供了WeakHashMap类，使用和这个类，它的键自然就是弱引用对象，无需我们再手动包装原始对象。这样一来，当productA变为null时（表明它所引用的Product已经无需存在于内存中），这时指向这个Product对象的就是由弱引用对象weakProductA了，那么显然这时候相应的Product对象时弱可达的，所以指向它的弱引用会被清除，这个Product对象随即会被回收，指向它的弱引用对象会进入引用队列中。

## 引用队列

下面我们来简单地介绍下引用队列的概念。实际上，WeakReference类有两个构造函数：

```java
//创建一个指向给定对象的弱引用
WeakReference(T referent) 
//创建一个指向给定对象并且登记到给定引用队列的弱引用
WeakReference(T referent, ReferenceQueue<? super T> q) 
```

我们可以看到第二个构造方法中提供了一个ReferenceQueue类型的参数，通过提供这个参数，我们便把创建的弱引用对象注册到了一个引用队列上，这样当它被垃圾回收器清除时，就会把它送入这个引用队列中，我们便可以对这些被清除的弱引用对象进行统一管理。

```java
public class ReferenceTest {

    private static ReferenceQueue<VeryBig> rq = new ReferenceQueue<VeryBig>();

    public static void checkQueue() {
        Reference<? extends VeryBig> ref = null;
        while ((ref = rq.poll()) != null) {
            if (ref != null) {
                System.out.println("In queue: "    + ((VeryBigWeakReference) (ref)).id);
            }
        }
    }

    public static void main(String args[]) {
        int size = 3;
        LinkedList<WeakReference<VeryBig>> weakList = new LinkedList<>();
        for (int i = 0; i < size; i++) {
            weakList.add(new VeryBigWeakReference(new VeryBig("Weak " + i), rq));
            System.out.println("Just created weak: " + weakList.getLast());

        }

        System.gc(); 
        try { // 下面休息几分钟，让上面的垃圾回收线程运行完成
            Thread.currentThread().sleep(6000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        checkQueue();
    }
}

class VeryBig {
    public String id;
    // 占用空间,让线程进行回收
    byte[] b = new byte[2 * 1024];

    public VeryBig(String id) {
        this.id = id;
    }

    protected void finalize() {
        System.out.println("Finalizing VeryBig " + id);
    }
}

class VeryBigWeakReference extends WeakReference<VeryBig> {
    public String id;

    public VeryBigWeakReference(VeryBig big, ReferenceQueue<VeryBig> rq) {
        super(big, rq);
        this.id = big.id;
    }

    protected void finalize() {
        System.out.println("Finalizing VeryBigWeakReference " + id);
    }
}
```

最后的输出结果为：

```java
Just created weak: com.javabase.reference.VeryBigWeakReference@1641c0
Just created weak: com.javabase.reference.VeryBigWeakReference@136ab79
Just created weak: com.javabase.reference.VeryBigWeakReference@33c1aa
Finalizing VeryBig Weak 2
Finalizing VeryBig Weak 1
Finalizing VeryBig Weak 0
In queue: Weak 1
In queue: Weak 2
In queue: Weak 0
```

# 虚引用（PhantomReference）

  “虚引用”顾名思义，就是形同虚设，与其他几种引用都不同，虚引用并不会决定对象的生命周期。如果一个对象仅持有虚引用，那么它就和没有任何引用一样，在任何时候都可能被垃圾回收器回收。

  虚引用主要用来跟踪对象被垃圾回收器回收的活动。虚引用与软引用和弱引用的一个区别在于：虚引用必须和引用队列 （ReferenceQueue）联合使用。当垃圾回收器准备回收一个对象时，如果发现它还有虚引用，就会在回收对象的内存之前，把这个虚引用加入到与之 关联的引用队列中。

**5、总结**

  Java4种引用的级别由高到低依次为：

  **强引用  >  软引用  >  弱引用  >  虚引用**

  通过图来看一下他们之间在垃圾回收时的区别：

   ![image-20200414222838274](D:\document\record\study-note\java\java引用相关.assets\image-20200414222838274.png)

​    当垃圾回收器回收时，某些对象会被回收，某些不会被回收。垃圾回收器会从根对象Object来标记存活的对象，然后将某些不可达的对象和一些引用的对象进行回收，如果对这方面不是很了解，可以参考如下的文章：

   通过表格来说明一下，如下：

![image-20200414222724100](assets\image-20200414222724100.png)