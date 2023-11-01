# 简介

MVCC，Multi-Version Concurrency Control，多版本并发控制。MVCC 是一种并发控制的方法，一般在数据库管理系统中，实现对数据库的并发访问中的无锁并发（我个人叫做无阻塞并发）；

通常我们在实现并发控制的时候**最简单的方式是使用加锁，读写完全互斥的方式来实现**，但是这样就意味着，**事务A在写的时候，会阻塞所有的读事务的操作，这对系统吞吐量，以及性能的损耗是相当大的**。所以就有了MVCC，它可以在一个事务A写这条数据的情况下，实现其他事务还可以读这条数据。**实现了一致性非阻塞读**

**缺点**就是：这些快照数据会占用更多的空间，同时也需要定期清理（系统自动），**也就是采用了空间换时间的思路**

# MVCC实现原理

## 基本概念

在MySQL中建表时，每个表记录都会有**三列隐藏记录**，其中**和MVCC有关系的有两列**

* DB_TRX_ID   数据行版本号

* DB_ROLL_PT 回滚指针（指向undo log数据用于回滚）

  > 后续内容图解中会有更清晰的图形解释该字段

## mvcc 依赖项

MVCC 在mysql 中的实现依赖的是 **undo log** 与 [**read view**](#read view)。

### undo log

 undo log中记录的是数据表记录行的多个版本，也就是事务执行过程中的回滚段,其实就是MVCC 中的一行原始数据的多个版本镜像数据。

#### undo 行数据更新过程

初始数据：table name : test

![image-20200502153934586](assets\image-20200502153934586.png)

第一次修改数据 ：

```sql
 update test set testId=20 where id = 1;
```

![image-20200502154249884](assets\image-20200502154249884.png)

第二次修改数据 ：

```
 update test set testId=30 where id = 1;
```

![image-20200502154623061](assets\image-20200502154623061.png)

### read view

系统当前的 read view 数据：

**m_ids**：在⽣成ReadView时，当前系统中活跃的事务id列表

**min_trx_id**：在⽣成ReadView时，当前系统中活跃的最⼩的事务id，也就是m_ids中的最⼩值

**max_trx_id**：在⽣成ReadView时，系统应该分配给下⼀个事务的事务id值

**creator_trx_id**：⽣成该ReadView的事务的事务id



可见性判断的伪代码：不可见的 trx_id 都会根据版本链继续使用这个方法查找到可见的为止否则查询直到链的最末端。

```
IsVisible(trx_id)
    if (trx_id == creator_trx_id)     // 当前事务
        return true;
    else if (trx_id < min_trx_id)    // ReadView创建时, 事务已提交
        return true;
    else if (trx_id >= max_trx_id)  // ReadView创建时，事务还未被创建
        return false;
    else if (trx_id is in m_ids)  // ReadView创建时，事务正在执行，但未提交,也就是活跃中的事务
        return false
    else                          // ReadView创建时, 事务已提交
        return true;
```

## 快照数据

事务的第一个DML语句执行时，会去建立当前表的数据的快照，该数据中可能包含了未提交的数据，**这些数据都有undo log 指针指向之前版本的数据**