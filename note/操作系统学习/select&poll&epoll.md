



看这个：

1. https://mp.weixin.qq.com/s/CMWlDywI1zbgJSoeGTBmuw
2. https://blog.csdn.net/weixin_39312465/article/details/86385419
3. https://zhuanlan.zhihu.com/p/272891398

**补一嘴：epoll 使用到了 mmap 来让用户程序和内核共享这个维护 epoll的数据结构的内存，这样就避免了维护在其中的文件描述符 从用户态到内核态的拷贝性能消耗。**

