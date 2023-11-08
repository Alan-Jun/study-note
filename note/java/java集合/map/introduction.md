## Map

* **non thread safety** 非线程安全

  * TreeMap：基于红黑树实现。
  * HashMap：基于哈希表实现。

  * LinkedHashMap：使用双向链表来维护元素的顺序，顺序为插入顺序或者最近最少使用（LRU）顺序。存储结构采用Hashmap的存储数据结构，

* **thread safety** 线程安全
  * HashTable：和 HashMap 类似，但它是线程安全的，这意味着同一时刻多个线程同时写入 HashTable 不会导致数据不一致。它是遗留类，不应该去使用它，而是使用 ConcurrentHashMap 来支持线程安全，ConcurrentHashMap 的效率会更高，因为 1.8  ConcurrentHashMap 数组节点使用CAS自旋更新，而链表上使用 synchronized 来同步，并发的时候锁的粒度更细，并发性能更好。
  
  * [ConcurrentHashMap](thread-safety/ConcurrentHashMap.md)
  
  * ConcurrentSkipListMap
  
    