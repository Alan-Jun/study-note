# CopyOnWriteArrayList

`CopyOnWriteArrayList`是为并发情况下设计的一个 list实现，它主要增强的并发读写时候的读的性能，利用了版本管理的方式来实现读写并行，这样带来的问题就是你可能读取到历史数据，然后是写与写之间会使用 `ReentrantLock`锁来控制并发。

下面我们来大致看一下它的实现方式，这里主要抽几个典型的方式作为介绍，因为其他都是一样的

```java
public class CopyOnWriteArrayList<E>
    implements List<E>, RandomAccess, Cloneable, java.io.Serializable {
    private static final long serialVersionUID = 8673264195747942595L;

    /** The lock protecting all mutators */
    final transient ReentrantLock lock = new ReentrantLock();

    /** The array, accessed only via getArray/setArray. */
    private transient volatile Object[] array;
    
    final Object[] getArray() {
        return array;
    }

    final void setArray(Object[] a) {
        array = a;
    }
    
    public CopyOnWriteArrayList() {
        setArray(new Object[0]);
    }

    public CopyOnWriteArrayList(Collection<? extends E> c) {
        Object[] elements;
        if (c.getClass() == CopyOnWriteArrayList.class)
            elements = ((CopyOnWriteArrayList<?>)c).getArray();
        else {
            elements = c.toArray();
            // c.toArray might (incorrectly) not return Object[] (see 6260652)
            if (elements.getClass() != Object[].class)
                elements = Arrays.copyOf(elements, elements.length, Object[].class);
        }
        setArray(elements);
    }

    public CopyOnWriteArrayList(E[] toCopyIn) {
        setArray(Arrays.copyOf(toCopyIn, toCopyIn.length, Object[].class));
    }
    
    public int size() {
        return getArray().length;
    }

    public boolean isEmpty() {
        return size() == 0;
    }
}
```

> 需要注意的是这里使用到的标记接口 `RandomAccess` 他的作用其实是用来利用`instanceof`来判断哪一个类实现了`RandomAccess`
>
> 比如 `ArrayList` 和 `LinedList` 其中`ArrayList` 就实现了这个`RandomAccess` 接口，表示它的随机访问快，因为是数组实现的，这时候比如我们在做便利的时候就能根据这个值来判断了，`RandomAccess` 使用for(int i=0;i.....)循环 比较快，非`RandomAccess`使用它的迭代器便利会更快，比如`LinedList`,要是使用fori这种循环，通过get(i)访问那就很慢，需要因为每一次get(i)都需要在链表中做链路查找

来看两个写的方法`add`,`remover`，以及读方法`get`,`indexOf`，可以看到都是加了锁的，同时采用了写时复制的思想，这样不回去修改到原来的那块内存，读的时候就能去正常读取那块数据了，缺点就是在没有重新修改 array 指针前，读取到的是历史数据

**add(e)**

```java
public boolean add(E e) {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        Object[] elements = getArray();
        int len = elements.length;
        Object[] newElements = Arrays.copyOf(elements, len + 1);
        newElements[len] = e;
        setArray(newElements);
        return true;
    } finally {
        lock.unlock();
    }
}
```

**remove(i)**

```java
public E remove(int index) {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        Object[] elements = getArray();
        int len = elements.length;
        E oldValue = get(elements, index);
        int numMoved = len - index - 1;
        if (numMoved == 0)
            setArray(Arrays.copyOf(elements, len - 1));
        else {
            Object[] newElements = new Object[len - 1];
            System.arraycopy(elements, 0, newElements, 0, index);
            System.arraycopy(elements, index + 1, newElements, index,
                             numMoved);
            setArray(newElements);
        }
        return oldValue;
    } finally {
        lock.unlock();
    }
}
```

**get(i)**

```java
public E get(int index) {
    return get(getArray(), index);
}

private E get(Object[] a, int index) {
    return (E) a[index];
}
```

**indexOf(o)**

```java
public int indexOf(Object o) {
    Object[] elements = getArray();
    return indexOf(o, elements, 0, elements.length);
}

private static int indexOf(Object o, Object[] elements,int index, int fence) {
    if (o == null) {
        for (int i = index; i < fence; i++)
            if (elements[i] == null)
                return i;
    } else {
        for (int i = index; i < fence; i++)
            if (o.equals(elements[i]))
                return i;
    }
    return -1;
}
```