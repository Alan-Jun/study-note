# AtomicStampedReference

stamp  可以用来解决ABA问题（有了 stamp  就算最后的数据相同，我们也可以通过stamp 知道它是不同的值），同时还可以记录被修改了多少次

# 基本结构代码

```java
public class AtomicStampedReference<V> {
    //内部维护一个静态内部类
    //reference表示引用对象
    //stamp表示时间戳（版本号），int型 注意是 final的
    private static class Pair<T> {
        final T reference;
        final int stamp;
        private Pair(T reference, int stamp) {
            this.reference = reference;
            this.stamp = stamp;
        }
        static <T> Pair<T> of(T reference, int stamp) {
            return new Pair<T>(reference, stamp);
        }
    }

    private volatile Pair<V> pair;

    /**
     *  初始化方法
     */
    public AtomicStampedReference(V initialRef, int initialStamp) {
        pair = Pair.of(initialRef, initialStamp);
    }
    
    public boolean compareAndSet(V   expectedReference,
                                 V   newReference,
                                 int expectedStamp,
                                 int newStamp) {
        Pair<V> current = pair;
        //通过增加了stamp（版本号）的CAS操作，可以避免ABA问题，即更新始终是递增的，不会出现往复。
        return
            expectedReference == current.reference &&
            expectedStamp == current.stamp &&
            ((newReference == current.reference &&
              newStamp == current.stamp) ||
             casPair(current, Pair.of(newReference, newStamp)));// 修改成最新的 Pair
    }
    
    private boolean casPair(Pair<V> cmp, Pair<V> val) {
        return UNSAFE.compareAndSwapObject(this, pairOffset, cmp, val); // CAS修改 current pair 的 reference 指向最新的 reference
    }
}
```