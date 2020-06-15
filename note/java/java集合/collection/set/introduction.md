非线程安全类

- TreeSet：底层是[TreeMap](../../map/non-thread-safety/TreeMap.md)
- HashSet：底层是[hashMap](../../map/non-thread-safety/HashMap.md)
- LinkedHashSet：具有 HashSet 的查找效率，并且内部使用双向链表维护元素的插入顺序。

线程安全类

- CopyOnWriteArraySet : 底层是[CopyOnWriteArrayList](../list/thread-safety/CopyOnWriteArrayList.md)
- ConcurrentSkipListSet : 底层是[ConcurrentSkipListMap](../../map/thread-safety/ConcurrentSkipListMap.md)