# AtomicIntegerFieldUpdater

原子更新引用类型里的字段。

# 基本使用

```java
public class AtomicIntegerFieldUpdaterTest {
     private static Class<Person> cls;
     /**
      * AtomicIntegerFieldUpdater说明
      * 基于反射的实用工具，可以对指定类的指定 volatile int 字段进行原子更新。此类用于原子数据结构，
      * 该结构中同一节点的几个字段都独立受原子更新控制。
      * 注意，此类中 compareAndSet 方法的保证弱于其他原子类中该方法的保证。
      * 因为此类不能确保所有使用的字段都适合于原子访问目的，所以对于相同更新器上的 compareAndSet 和 set 的其他调用，
      * 它仅可以保证原子性和可变语义。
      * @param args
      */
     public static void main(String[] args) {
        // 新建AtomicLongFieldUpdater对象，传递参数是“class对象”和“long类型在类中对应的名称”
        AtomicIntegerFieldUpdater<Person> mAtoLong = AtomicIntegerFieldUpdater.newUpdater(Person.class, "id");
        Person person = new Person(12345);
        mAtoLong.compareAndSet(person, 12345, 1000);
        System.out.println("id="+person.getId());
     }
}


class Person {
    volatile int id;// 该字段必须是volatile的
    public Person(int id) {
        this.id = id;
    }
    public void setId(int id) {
        this.id = id;
    }
    public int getId() {
        return id;
    }
}
```

# 基本结构源码

```java
@CallerSensitive
public static <U> AtomicIntegerFieldUpdater<U> newUpdater(Class<U> tclass,String fieldName) {
    return new AtomicIntegerFieldUpdaterImpl<U>(tclass, fieldName, Reflection.getCallerClass());
}
 private static final class AtomicIntegerFieldUpdaterImpl<T>
        extends AtomicIntegerFieldUpdater<T> {
        private static final sun.misc.Unsafe U = sun.misc.Unsafe.getUnsafe();
     	// 维护的类的字段的内存偏移量
        private final long offset;
        /**
         * if field is protected, the subclass constructing updater, else
         * the same as tclass
         */
        private final Class<?> cclass;
        /** class holding the field */
        private final Class<T> tclass;

        AtomicIntegerFieldUpdaterImpl(final Class<T> tclass,
                                      final String fieldName,
                                      final Class<?> caller) {
            final Field field;
            final int modifiers;
            try {
                field = AccessController.doPrivileged(
                    new PrivilegedExceptionAction<Field>() {
                        public Field run() throws NoSuchFieldException {
                            return tclass.getDeclaredField(fieldName);
                        }
                    });
                modifiers = field.getModifiers();
                sun.reflect.misc.ReflectUtil.ensureMemberAccess(
                    caller, tclass, null, modifiers);
                ClassLoader cl = tclass.getClassLoader();
                ClassLoader ccl = caller.getClassLoader();
                if ((ccl != null) && (ccl != cl) &&
                    ((cl == null) || !isAncestor(cl, ccl))) {
                    sun.reflect.misc.ReflectUtil.checkPackageAccess(tclass);
                }
            } catch (PrivilegedActionException pae) {
                throw new RuntimeException(pae.getException());
            } catch (Exception ex) {
                throw new RuntimeException(ex);
            }

            if (field.getType() != int.class)
                throw new IllegalArgumentException("Must be integer type");

            if (!Modifier.isVolatile(modifiers))
                throw new IllegalArgumentException("Must be volatile type");

            // Access to protected field members is restricted to receivers only
            // of the accessing class, or one of its subclasses, and the
            // accessing class must in turn be a subclass (or package sibling)
            // of the protected member's defining class.
            // If the updater refers to a protected field of a declaring class
            // outside the current package, the receiver argument will be
            // narrowed to the type of the accessing class.
            this.cclass = (Modifier.isProtected(modifiers) &&
                           tclass.isAssignableFrom(caller) &&
                           !isSamePackage(tclass, caller))
                          ? caller : tclass;
            this.tclass = tclass;
            this.offset = U.objectFieldOffset(field);
        }
     
     // 省略.........................部分代码
 }

```

# 数据查询修改

方法具体实现，都在 AtomicIntegerFieldUpdaterImpl 中，和其他原子类的实现都是一样的。就不做过多说明，这里把代码列出来，只是为了，了解一下这个类具体有些什么功能，方便使用

```java
    public abstract boolean compareAndSet(T obj, int expect, int update);

    /**
     */
    public abstract boolean weakCompareAndSet(T obj, int expect, int update);

    /**
     * @param newValue the new value
     */
    public abstract void set(T obj, int newValue);

    /**
     * @since 1.6
     */
    public abstract void lazySet(T obj, int newValue);

    /**
     * @return the current value
     */
    public abstract int get(T obj);

    /**
     * @return the previous value
     */
    public int getAndSet(T obj, int newValue) {
        int prev;
        do {
            prev = get(obj);
        } while (!compareAndSet(obj, prev, newValue));
        return prev;
    }

    /**
     * @return the previous value
     */
    public int getAndIncrement(T obj) {
        int prev, next;
        do {
            prev = get(obj);
            next = prev + 1;
        } while (!compareAndSet(obj, prev, next));
        return prev;
    }

    /**
     * @return the previous value
     */
    public int getAndDecrement(T obj) {
        int prev, next;
        do {
            prev = get(obj);
            next = prev - 1;
        } while (!compareAndSet(obj, prev, next));
        return prev;
    }

    /**
     * @return the previous value
     */
    public int getAndAdd(T obj, int delta) {
        int prev, next;
        do {
            prev = get(obj);
            next = prev + delta;
        } while (!compareAndSet(obj, prev, next));
        return prev;
    }

    /**
     * @return the updated value
     */
    public int incrementAndGet(T obj) {
        int prev, next;
        do {
            prev = get(obj);
            next = prev + 1;
        } while (!compareAndSet(obj, prev, next));
        return next;
    }

    /**
     * @return the updated value
     */
    public int decrementAndGet(T obj) {
        int prev, next;
        do {
            prev = get(obj);
            next = prev - 1;
        } while (!compareAndSet(obj, prev, next));
        return next;
    }

    /**
     * @return the updated value
     */
    public int addAndGet(T obj, int delta) {
        int prev, next;
        do {
            prev = get(obj);
            next = prev + delta;
        } while (!compareAndSet(obj, prev, next));
        return next;
    }

    /**
     * @since 1.8
     */
    public final int getAndUpdate(T obj, IntUnaryOperator updateFunction) {
        int prev, next;
        do {
            prev = get(obj);
            next = updateFunction.applyAsInt(prev);
        } while (!compareAndSet(obj, prev, next));
        return prev;
    }

    /**
     * @since 1.8
     */
    public final int updateAndGet(T obj, IntUnaryOperator updateFunction) {
        int prev, next;
        do {
            prev = get(obj);
            next = updateFunction.applyAsInt(prev);
        } while (!compareAndSet(obj, prev, next));
        return next;
    }

    /**
     * @since 1.8
     */
    public final int getAndAccumulate(T obj, int x,
                                      IntBinaryOperator accumulatorFunction) {
        int prev, next;
        do {
            prev = get(obj);
            next = accumulatorFunction.applyAsInt(prev, x);
        } while (!compareAndSet(obj, prev, next));
        return prev;
    }

    /**
     * @since 1.8
     */
    public final int accumulateAndGet(T obj, int x,
                                      IntBinaryOperator accumulatorFunction) {
        int prev, next;
        do {
            prev = get(obj);
            next = accumulatorFunction.applyAsInt(prev, x);
        } while (!compareAndSet(obj, prev, next));
        return next;
    }

```

