# 在并发执行事务时会发生什么问题呢？

1. **脏读**：一个事务读到另一个事务未提交的更新数据（事务A和B并发执行，B事务执行更新后，A事务查询B事务没有提交的数据，B事务回滚，则A事务得到的数据不是数据库中的真实数据。也就是脏数据，即和数据库中不一致的数据）。
2. **不可重复读**：一是指在一个事务内，多次读同一数据。在这个事务还没有结束时，另外一个事务也访问该同一数据。那么，在第一个事务中的两次读数据之间，由于第二个事务的修改，那么第一个事务两次读到的的数据可能是不一样的。这样在一个事务内两次读到的数据是不一样的，因此称为是不可重复读。
3. **覆盖更新**：这是不可重复读中的特例，一个事务覆盖另一个事务已提交的更新数据（即A事务更新数据，然后B事务更新该数据，A事务查询发现自己更新的数据变了）。
4. **幻读**：是指当事务不是独立执行时发生的一种现象，例如第一个事务对一个表中的数据进行了修改，这种修改涉及到表中的全部数据行，同时，第二个事务也修改这个表中的数据，这种修改是向表中插入一行新数据。那么，以后就会发生操作第一个事务的用户发现表中还有没有修改的数据行，就好象发生了幻觉一样。

# 隔离级别

* Serializable (序列化，串行化) : 串行执行 **可避免脏读，不可重复读，幻读的发生。**它是最高的事务隔离级别，同事花费的代价也是很大的，性能很低，一般很少使用。

* Repeatable read (重复读) :一个事务开始读取这条数据，那么别的事务就不能对其进行修改。 可以解决不可重复读和脏读

* Read committed (读已提交): 一个事物要等另一个事务提交之后才能读取 可避免脏读 

* Read uncommitted(读未提交) 

  **mysql 默认是 `REPEATABLE-READ` 重复读**

# 如可查看级别

* 查看当前会话隔离级别

  `select @@tx_isolation;`

* 查看系统当前隔离级别

  `select @@global.tx_isolation;`

* 设置当前会话隔离级别

  ```sql
  set session transaction isolation level read uncommitted; 
  set session transaction isolation level read committed;
  set session transaction isolation level repeatable read;
  set session transaction isolation level serializable;
  ```

* 设置系统当前隔离级别

  ```
  set global  transaction isolation level read uncommitted; 
  set global  transaction isolation level read committed;
  set global  transaction isolation level repeatable read;
  set global  transaction isolation level serializable;
  ```

# 四个隔离界别如何实现的

## 须知

**我们的事务ID是在一个事务中的第一个语句执行时生成的**

> 下文中的 read view ，还有快照数据 相关的东西看[MVCC](MVCC.md)
>
> 锁相关：[mysql锁类型](mysql锁类型.md)

## read uncommitted(读未提交) 

**锁层面**：写数据都会加排他锁，普通select数据是不加锁的（除了serializable 级别以外），但是写写因为都加了锁所以可以避免并发写安全问题

**普通同不加锁，所以读写可以并行，读最新的数据，不管这条记录是不是已提交。不会遍历版本链，少了查找可见的版本的步骤**

## read committed (读已提交)

**锁层面**：写数据都会加排他锁，普通select数据是不加锁的（除了serializable 级别以外），但是写写因为都加了锁所以可以避免并发写安全问题，使用 mvcc +Read View 实现 读提交级别的无锁并发

**read view & 快照数据**： 会在每一个 select语句执行的时候都会去重新拉一次数据

查询的时候会遍历快照数据中的版本链表 通过 read view [判断数据对当前的事务的可见性](#read_view_sees_trx_id) 不可见就通过回滚指针找之前版本的数据 undo log

然后我们看下面的例子实际查询例子来证明一下上面的判断

1. 写事务在前， 读事务在后的（主要看是否会读未提交，以及重复度，幻读问题）(读事务ID小于写事务的情况)

   * 写（删除操作）未提交，开始读，然后写提交后再读一次

     ```sql
     -- 写事务 session A
     begin;
     delete from policyinfo where itemId = 199655;
     select sleep(10);
     commit;
     
     -- 读事务session B(第一次读 获取一个read view 和 当前的数据快照，按照上文判断逻辑，发现时未提交数据所以，读上一个版本数据，第二次重新拉，发现数据提交了，读取提交后的数据)
     begin; 
     SELECT * FROM policyinfo where itemId = 199655;
     select sleep(10);
     SELECT * FROM policyinfo where itemId = 199655;
     commit;
     
     -- 读事务 session B(第二次重复查询读到了提交的数据，这种不加锁的快照读不能解决幻读)
     begin; 
     SELECT count(*) FROM policyinfo where itemId >= 199655;
     select sleep(10);
     SELECT count(*) FROM policyinfo where itemId >= 199655;
     commit;
     
     -- 读事务（当前读，加了排他锁） session B 这种情况我需要会去安装一个mysql测试一下
     begin; 
     SELECT count(*) FROM policyinfo where itemId >= 199655 for update;
     select sleep(10);
     SELECT count(*) FROM policyinfo where itemId >= 199655 for update;
     commit;
     ```

2. 读事务在前，写事务在后

   * 读事务先sleep（20），写事务写提交，读事务开始读数据

     ```sql
     -- 读事务 session A（先开始的读事务后面可以读取到已提交的数据，因为read view是在每一次select语句建立的最新快照数据）
     begin; 
     SELECT * FROM policyinfo where itemId = 199655;
     select sleep(20);
     SELECT * FROM policyinfo where itemId = 199655;
     commit;
     
     -- 读事务（当前读，加了S锁） session A （由于读写互斥，让读事务在未提交前一致阻塞着写事务，达到了解决不可重复读，幻读的问题）
     begin; 
     SELECT * FROM policyinfo where itemId = 199655 lock in share mode ;
     select sleep(20);
     SELECT * FROM policyinfo where itemId = 199655;
     commit;
     
     -- 写事务 session B
     begin;
     update policyinfo set changingCondition=0 where itemId = 199655;
     commit;
     
     -- 写事务 session B
     begin;
     delete from policyinfo where itemId = 199655;
     commit;
     ```

**结论：**

read commit 级别 ，通过每次查询获取最新的read view 和最新的数据的快照，解决了脏读问题

如果使用 加锁的读方式（fro update / lock in share mode ） ,由于写写，读写互斥，所以写那边得不到提交，那么就可以解决不可重复读和幻读，脏读的问题，其实这也是 serializable  主动做的（会将所有select 都加 S锁 lock in share mode，任何级别写都是会自动X锁的）

## repeatable read (重复读)

**锁层面**：写数据都会加排他锁，普通select数据是不加锁的（除了serializable 级别以外），但是写写因为都加了锁所以可以避免并发写安全问题，使用 mvcc + Read View 实现 读提交级别的无锁并发

**read view & 快照数据**： 会在每一个事务的第一个select语句执行的时候都会只去拉一次数据

查询的时候会遍历`快照数据`通过 read view [判断数据对当前的事务的可见性](#read_view_sees_trx_id) 不可见就通过回滚指针找之前版本的数据 undo log

1. 写事务在前，读事务在后（读事务版本小于写事务的情况）

   * 写（删除操作）未提交，开始读，然后写提交后再读一次

     ```sql
     -- 写事务 session A
     begin;
     delete from policyinfo where itemId = 199655;
     select sleep(10);
     commit;
     
     -- 读事务 session B（第一次读取建立这个数据中这个读的唯一read view,以及取了该表的快照数据，查询数据取得数据的read_view_sees_trx_id函数判断这条数据不可见，读取undo log,第二次读使用的是第一次建立的read view所以判断结果相同）
     begin; 
     SELECT * FROM policyinfo where itemId = 199655;
     select sleep(10);
     SELECT * FROM policyinfo where itemId = 199655;
     commit;
     
     -- 读事务 session B (和上面基本相同，由于只取了一次热爱的 read view ,表的快照数据，所以第二次读页感知不到写事务的数据提交)
     begin; 
     SELECT count(*) FROM policyinfo where itemId >= 199655;
     select sleep(10);
     SELECT count(*) FROM policyinfo where itemId >= 199655;
     commit;
     ```

2. 读事务在前，写事务在后

   * 读事务先sleep(20），写事务写提交，读事务开始读数据

     ```sql
     -- 读事务（能够读取到后面的写事务提交的数据，因为那时候读事务select语句执行时，才建立的read view，和表数据快照（存在此时写事务修改的最新数据），read view中不存在写事务的未提交的记录，read_view_sees_trx_id函数判断数据可见）
     begin; 
     select sleep(20);
     SELECT * FROM policyinfo where itemId = 199655;
     commit;
     
     -- 写事务
     begin;
     update policyinfo set changingCondition=0 where itemId = 199655;
     commit;
     
     -- 写事务
     begin;
     delete from policyinfo where itemId = 199655;
     commit;
     ```

   * 读事务先读一次然后sleep(15),写事务写提交，读事务再读

     ```sql
     --读事务（两次读取到的数据是一样的，因为第一次select 建立 read view,还有表数据快照（这个中没有后面的写事务修改的数据），read view 中也不会没有写事务对数据修改的事务id，不管怎样都不会受下面数据的影响）
     begin; 
     SELECT * FROM policyinfo where itemId = 199655;
     select sleep(15);
     SELECT * FROM policyinfo where itemId = 199655;
     commit;
     
     -- 写事务
     begin;
     update policyinfo set changingCondition=1 where itemId = 199655;
     commit;
     
     ```

     ```sql
     -- 读事务（和上面一样）
     begin; 
     SELECT count(*) FROM policyinfo where itemId >= 199655;
     select sleep(15);
     SELECT count(*) FROM policyinfo where itemId >= 199655;
     commit;
     
     -- 写事务
     begin;
     delete from policyinfo where itemId = 199655;
     commit;
     
     ```


**结论：**

read commit 级别 ， 会在每一个事务的第一个select语句执行的时候都会只去拉一次read view & 表快照数据（数据中只会存在建立该数据前，的数据样本呢），是的每次查询处理的数据都是相同的，也就解决了幻读，重复读，read view 中存放了 当时未提交的事务，也解决了脏读问题

需要注意的是：该级别解决的幻读，下面这种情况，由于写操作不是在快照数据上操作的，第二次读看到了数据行的新增

```sql
-- 读写事务
begin; 
SELECT count(*) FROM policyinfo where itemId >= 199655;
select sleep(15);
update policyinfo set changingCondition=0 where itemId = 199660;
SELECT count(*) FROM policyinfo where itemId >= 199655;
commit;

-- 写事务
begin;
insert into pkgproductdb.policyinfo values(199660, 1, 0, NULL
);
commit;
```

如果使用 加锁的读方式（fro update / lock in share mode ） ,由于写写，读写互斥，也可以做到解决幻读的，但是这又回到了上面说的，相当于使用了 serializable的处理方式

## serializable 级别

该级别下，会自动将所有普通`select`转化为`select ... lock in share mode`执行（加读的共享锁），即针对同一数据的所有读写都变成互斥的了，可靠性大大提高，并发性大大降低。

## read_view_sees_trx_id

```
IsVisible(trx_id)
    if (trx_id == creator_trx_id)     // 当前事务
        return true;
    else if (trx_id < up_limit_id)    // ReadView创建时, 事务已提交
        return true;
    else if (trx_id >= low_limit_id)  // ReadView创建时，事务还未被创建
        return false;
    else if (trx_id is in m_ids)  // ReadView创建时，事务正在执行，但未提交
        return false
    else                          // ReadView创建时, 事务已提交
        return true;
```



> https://blog.csdn.net/ashic/article/details/53735537
>
> https://www.zhihu.com/question/263820564