https://segmentfault.com/a/1190000003063859

http://blog.chinaunix.net/uid-26912934-id-3308700.html

https://github.com/CyC2018/CS-Notes/blob/master/notes/Socket.md#select



看这个：

https://blog.csdn.net/weixin_39312465/article/details/86385419

https://zhuanlan.zhihu.com/p/272891398

**补一嘴：epoll 使用到了 mmap 来让用户程序和内核共享这个维护 epoll的数据结构的内存，这样就避免了维护在其中的文件描述符 从用户态到内核态的拷贝性能消耗。**