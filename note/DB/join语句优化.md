

> https://my.oschina.net/xinxingegeya/blog/495897

Using join buffer (Block Nested Loop)

msyql的表连接算法



# Nested Loop Join(NLJ)算法

 NLJ 算法:将驱动表/外部表的结果集作为循环基础数据，然后循环从该结果集每次一条获取数据作为下一个表的过滤条件查询数据，然后合并结果。如果有多表join，则将前面的表的结果集作为循环数据，取到每行再到联接的下一个表中循环匹配，获取结果集返回给客户端。

Nested-Loop 的伪算法如下:

```
for each row in t1 matching range {
  for each row in t2 matching reference key {
    for each row in t3 {
      if row satisfies join conditions,
      send to client
    }
  }
}
```

Because the NLJ algorithm passes rows one at a time from outer loops to inner loops, tables processed in the inner loops typically are read many times

因为普通Nested-Loop一次只将一行传入内层循环, 所以外层循环(的结果集)有多少行, 内存循环便要执行多少次.在内部表的连接上有索引的情况下，其扫描成本为O(Rn),若没有索引,则扫描成本为O(Rn*Sn)。如果内部表S有很多记录，则SimpleNested-Loops Join会扫描内部表很多次，执行效率非常差。



# Block Nested-Loop Join(BNL)算法

BNL 算法:将外层循环的行/结果集存入join buffer, 内层循环的每一行与整个buffer中的记录做比较，从而减少内层循环的次数。

举例来说，外层循环的结果集是100行，使用NLJ 算法需要扫描内部表100次，如果使用BNL算法，先把对Outer Loop表(外部表)每次读取的10行记录放到join buffer,然后在InnerLoop表(内部表)中直接匹配这10行数据，内存循环就可以一次与这10行进行比较, 这样只需要比较10次，对内部表的扫描减少了9/10。所以BNL算法就能够显著减少内层循环表扫描的次数。

前面描述的query, 如果使用join buffer, 那么实际join示意如下:

```
for each row in t1 matching range {
  for each row in t2 matching reference key {
    store used columns from t1, t2 in join buffer
    if buffer is full {
      for each row in t3 {
        for each t1, t2 combination in join buffer {
          if row satisfies join conditions,
          send to client
        }
      }
      empty buffer
    }
  }
}

if buffer is not empty {
  for each row in t3 {
    for each t1, t2 combination in join buffer {
      if row satisfies join conditions,
      send to client
    }
  }
}
```

如果t1, t2参与join的列长度只和为s, c为二者组合数, 那么t3表被扫描的次数为 

> (S * C)/join_buffer_size + 1

扫描t3的次数随着join_buffer_size的增大而减少, 直到join buffer能够容纳所有的t1, t2组合,  再增大join buffer size, query 的速度就不会再变快了。

MySQL使用Join Buffer有以下要点:

>  \1. join_buffer_size变量决定buffer大小。
>
>  \2. 只有在join类型为all, index, range的时候才可以使用join buffer。
>
>  \3. 能够被buffer的每一个join都会分配一个buffer, 也就是说一个query最终可能会使用多个join buffer。
>
>  \4. 第一个nonconst table不会分配join buffer, 即便其扫描类型是all或者index。
>
>  \5. 在join之前就会分配join buffer, 在query执行完毕即释放。
>
>  \6. join buffer中只会保存参与join的列, 并非整个数据行。

如何使用 

5.6版本及以后，优化器管理参数optimizer_switch中中的block_nested_loop参数控制着BNL是否被用于优化器。默认条件下是开启，若果设置为off，优化器在选择 join方式的时候会选择NLJ算法。



**总结**

1. 如果使用Block Nested Loop Join算法的话，通过扩大一次缓存区的大小也能减小内循环的次数。**由此又可得，设置合理的缓冲区大小能够提高连接效率**

**2.快速匹配：**扫描被驱动表寻找合适的记录可以看做一个查询操作，如何提高查询的效率呢？建索引啊！**由此还可得出，在被驱动表建立索引能够提高连接效率**

**3.排序：**假设t1表驱动t2表进行连接操作，连接条件是t1.id=t2.id，而且要求查询结果对id排序。现在有两种选择，方式一[...ORDER BY t1.id]，方式二[...ORDER BY t2.id]。如果我们使用方式一的话，可以先对t1进行排序然后执行表连接算法，如果我们使用方式二的话，只能在执行表连接算法后，对结果集进行排序（Using temporary），效率自然低下。**由此最后可得出，优先选择驱动表的属性进行排序能够提高连接效率。**