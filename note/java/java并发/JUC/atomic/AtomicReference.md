# AtomicReference

原子更新引用类型。原理没什么好说的了，需要的就是了解一下它现在支持的所有方法,以及基本使用

# 基本使用

```java
public class AtomicReferenceTest {        
    public static AtomicReference<user> atomicUserRef = new AtomicReference<user>();  
    public static void main(String[] args) {                
        User user = new User("conan"， 15);                
        atomicUserRef.set(user);                
        User updateUser = new User("Shinichi"， 17);                
        atomicUserRef.compareAndSet(user， updateUser);             
        System.out.println(atomicUserRef.get().getName());       
        System.out.println(atomicUserRef.get().getOld());      
    }        
    static class User {         
        private String name;                
        private int old;               
        public User(String name， int old) {  
            this.name = name;         
            this.old = old;            
        }               
        public String getName() { 
            return name;            
        }            
        public int getOld() {  
            return old;
                
        }        
    } 
}
```

# 结构源码

```java
public class AtomicReference<V> implements java.io.Serializable {
    private static final long serialVersionUID = -1848883965231344442L;

    private static final Unsafe unsafe = Unsafe.getUnsafe();
    private static final long valueOffset;

    //class初始化时执行
    static {
        try {
            valueOffset = unsafe.objectFieldOffset
                (AtomicReference.class.getDeclaredField("value"));
        } catch (Exception ex) { throw new Error(ex); }
    }

    //通过volatile关键字保证value值的可见性。
    private volatile V value;
    
    // 下面是两个初始化方法
    
    public AtomicReference(V initialValue) {
        value = initialValue;
    }
    
    public AtomicReference() {
    }
}
```

# 数据查询修改方法

```java
/**
 * Gets the current value. 由 volatile 做可见性保证
 *
 * @return the current value
 */
public final V get() {
    return value;
}

/**
 * 非原子操作，不宜在存在并发安全的地方使用
 *
 * @param newValue the new value
 */
public final void set(V newValue) {
    value = newValue;
}

/**
 * 我们很多原子类中都有 lazySet这样的方法，lazy 就是在不需要让共享变量的修改立刻让其他线程可见的时候，以设置普通变量的方式来修改该变量，以减少不必要的内存
 * 屏障，从而提高程序执行的效率
 * @param newValue the new value
 * @since 1.6
 */
public final void lazySet(V newValue) {
    unsafe.putOrderedObject(this, valueOffset, newValue);
}


public final boolean compareAndSet(V expect, V update) {
    return unsafe.compareAndSwapObject(this, valueOffset, expect, update);
}

/**
 * 
 * 1.8 中该方法没有任何的特别之处，和 compareAndSet 是一样的效果，后面的版本应该会有变化
 */
public final boolean weakCompareAndSet(V expect, V update) {
    return unsafe.compareAndSwapObject(this, valueOffset, expect, update);
}


@SuppressWarnings("unchecked")
public final V getAndSet(V newValue) {
    return (V)unsafe.getAndSetObject(this, valueOffset, newValue);
}

/**
 * 相较于以前的方法，增加了1.8特有的，方法传值（也就是 lambda的基石）
 * @since 1.8
 */
public final V getAndUpdate(UnaryOperator<V> updateFunction) {
    V prev, next;
    do {
        prev = get();
        next = updateFunction.apply(prev);
    } while (!compareAndSet(prev, next));
    return prev;
}

/**
 * @since 1.8
 */
public final V updateAndGet(UnaryOperator<V> updateFunction) {
    V prev, next;
    do {
        prev = get();
        next = updateFunction.apply(prev);
    } while (!compareAndSet(prev, next));
    return next;
}

/**
 * @since 1.8
 */
public final V getAndAccumulate(V x,
                                BinaryOperator<V> accumulatorFunction) {
    V prev, next;
    do {
        prev = get();
        next = accumulatorFunction.apply(prev, x);
    } while (!compareAndSet(prev, next));
    return prev;
}

/**
 * @since 1.8
 */
public final V accumulateAndGet(V x,
                                BinaryOperator<V> accumulatorFunction) {
    V prev, next;
    do {
        prev = get();
        next = accumulatorFunction.apply(prev, x);
    } while (!compareAndSet(prev, next));
    return next;
}
```

