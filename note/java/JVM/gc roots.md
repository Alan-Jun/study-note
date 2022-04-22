Gc root

1. 虚拟机栈中引用的对象
2. 方法区中静态属性、常量引用的对象
3. 本地方法栈中引用的对象
4. 被Synchronized锁持有的对象
5. 记录当前被加载类的SystemDictionary
6. 记录字符串常量引用的StringTable
7. 存在跨代引用的对象
8. 和GC Root处于同一CardTable的对象
