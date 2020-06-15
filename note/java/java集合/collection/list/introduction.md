# List在java中的整体结构图

![image-20200531114927467](assets/image-20200531114927467.png)

# List接口介绍

- **non thread safety** 

  - [ArrayList]：基于动态数组实现，支持随机访问。

  - [LinkedList]：基于双向链表实现，只能顺序访问，但是可以快速地在链表中间插入和删除元素。不仅如此，LinkedList 还可以用作栈、队列和双向队列。
  - [Stack]

- **thread safety**

  - [CopyOnWriteArrayList](thread-safety/CopyOnWriteArrayList.md)
  
    CopyOnWriteArrayList规避了只读操作（如get/contains）并发的瓶颈，但是它为了做到这点，在修改操作中做了很多工作和修改可见性规则。 此外，修改/插入操作还会锁住整个List，并且所有的修改操作都会都会新建数组，然后再将数组新得引用set回去，也就是说每次修改/插入的代价都比较高，因此这也是一个并发瓶颈。所以从理论上来说，CopyOnWriteArrayList并不算是一个通用的并发List。
  
  - Vector：和 ArrayList 类似，但它是线程安全的。