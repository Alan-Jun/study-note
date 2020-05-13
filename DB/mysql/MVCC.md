# 简介

MVCC，Multi-Version Concurrency Control，多版本并发控制。MVCC 是一种并发控制的方法，一般在数据库管理系统中，实现对数据库的并发访问中的无锁并发（我个人叫做无阻塞并发）；

通常我们在实现并发控制的时候**最简单的方式是使用加锁，读写完全互斥的方式来实现**，但是这样就意味着，**事务A在写的时候，会阻塞所有的读事务的操作，这对系统吞吐量，以及性能的损耗是相当大的**。所以就有了MVCC，它可以在一个事务A写这条数据的情况下，实现其他事务还可以读这条数据。**实现了一致性非阻塞读**

**缺点**就是：浙西而快照数据会占用更多的空间，同时也需要定期清理（系统自动），**也就是采用了空间换时间的思路**

# MVCC实现原理

## 基本概念

在MySQL中建表时，每个表记录都会有**三列隐藏记录**，其中**和MVCC有关系的有两列**

* DB_TRX_ID   数据行版本号

* DB_ROLL_PT 回滚指针（指向undo log数据用于回滚）

  > 后续内容图解中会有更清晰的图形解释该字段

## mvcc 依赖项

MVCC 在mysql 中的实现依赖的是 **undo log** 与 **read view**。

1. undo log: undo log中记录的是数据表记录行的多个版本，也就是事务执行过程中的回滚段,其实就是MVCC 中的一行原始数据的多个版本镜像数据。
2. read view: 主要用来判断当前版本数据的可见性。

## undo 行数据更新过程

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

## read view

### 基础介绍

read view 数据存放的是当前的未commit的活跃事务列表数据

### 判断当前版本数据项是否可见规则

使用了 read_view_sees_trx_id函数，我们的read_view中保存了当前全局的事务的范围：
`[up_limit_id，low_limit_id ]`

**1. 当行记录的事务ID小于当前系统的最小活动id，就是可见的。**
　　if (trx_id < view->up_limit_id) {
　　　　return(TRUE);
　　}
**2. 当行记录的事务ID大于当前系统的最大活动id，就是不可见的。**
　　if (trx_id >= view->low_limit_id) {
　　　　return(FALSE);
　　}
**3. 当行记录的事务ID在活动范围之中时，判断是否在活动链表中，如果在就不可见，如果不在就是可见的。**
　　for (i = 0; i < n_ids; i++) {
　　　　trx_id_t view_trx_id
　　　　　　= read_view_get_nth_trx_id(view, n_ids - i - 1);
　　　　if (trx_id <= view_trx_id) {
　　　　return(trx_id != view_trx_id);
　　　　}
　　}

## MVCC-查询规则

检验该行记录的当前数据`DB_TRX_ID数据行版本号`<=`当前事务版本号`，如果小于就直接返回数据，大于就根据 `DB_ROLL_PT 回滚指针`找到上一个版本数据（undo log）,继续判断版本，直到找到数据版本号<=当前事务版本号的数据返回 , 这样也就确保了读取到的数据是当前事务开始前已经存在的数据，或者是自身事务改变过的数据

