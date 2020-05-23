容器主要包括 Collection 和 Map 两种，Collection 存储着对象的集合，而 Map 存储着键值对（两个对象）的映射表。

- [Collection](collection/introduction.md)
- [Map](map/introduction.md)

集合使用中 我们有的时候会用到排序，比较什么的，这时候就有两个重要的的接口 

* Comparable 

  需要比较的元素只要实现这个接口的 `public int compareTo(T o); ` 这个方法主要就是判断当前实例和 `o`的大小关系。你可以用它灵活的定义大小关系。来实现排序，比较功能

* Comparator 比较器，需要用到比较的集合，我们需要给它设置一个比较器，需要实现 `int compare(T o1, T o2);`  这个也就是判断 o1 和o2 的大小关系，可以用它灵活的定义大小关系。来实现排序，比较功能

  比如我们在使用  PriorityQueue  的时候，因为它的实现就是默认最小堆的实现，如果我们想获得最大堆，那么在 按照正常 o1>o2  返回 >0, o1=o2  返回 0, o1<o2  返回 <0,用来使用最小堆没问题，但是要用 PriorityQueue 实现最大堆的话，我们就要颠倒这个关系 o1>o2  返回 <0, o1=o2  返回 0, o1<o2  返回 >0.这样就能得到一个最大堆

java中的并发安全集合类绝对安全吗？

不是的，并发安全集合类就和我们的DB一样的，如果我们多线程采取都改写操作的话，也会出现数据安全问题，比如ConcurrentHashMap

```java
public class CHMDemo {
    public static void main(String[] args) throws InterruptedException {
        ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<String,Integer>();
        map.put("key", 1);
        ExecutorService executorService = Executors.newFixedThreadPool(100);
        for (int i = 0; i < 1000; i++) {
            executorService.execute(new Runnable() {
                @Override
                public void run() {// 这里做读改写操作
                    int key = map.get("key") + 1; //step 1
                    map.put("key", key);//step 2
                }
            });
        }
        Thread.sleep(3000); //模拟等待执行结束
        System.out.println("------" + map.get("key") + "------");
        executorService.shutdown();
    }
}
```

读和读是可以并行的，那么就会读取到相同的值，修改，然后写入就发生了数据不一致问题。

这里解决的方法是 我们可以用原子类

```java
public class CHMDemo {
    public static void main(String[] args) throws InterruptedException {
        ConcurrentHashMap<String, AtomicInteger> map = new ConcurrentHashMap<String,AtomicInteger>();
        AtomicInteger integer = new AtomicInteger(1);
        map.put("key", integer);
        ExecutorService executorService = Executors.newFixedThreadPool(100);
        for (int i = 0; i < 1000; i++) {
            executorService.execute(new Runnable() {
                @Override
                public void run() {// 读改写
                    map.get("key").incrementAndGet();
                }
            });
        }// 都改写
        Thread.sleep(3000); //模拟等待执行结束
        System.out.println("------" + map.get("key") + "------");
        executorService.shutdown();
    }
}
```

 