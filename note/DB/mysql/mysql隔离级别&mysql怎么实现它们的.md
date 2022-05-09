# 在并发执行事务时会发生什么问题呢？

1. **脏读**：一个事务读到另一个事务未提交的更新数据（事务A和B并发执行，B事务执行更新后，A事务查询B事务没有提交的数据，B事务回滚，则A事务得到的数据不是数据库中的真实数据。也就是脏数据，即和数据库中不一致的数据）。
2. **不可重复读**：一是指在一个事务内，多次读同一数据。在这个事务还没有结束时，另外一个事务也访问该同一数据。那么，在第一个事务中的两次读数据之间，由于第二个事务的修改，那么第一个事务两次读到的的数据可能是不一样的。这样在一个事务内两次读到的数据是不一样的，因此称为是不可重复读。
3. **覆盖更新**：这是不可重复读中的特例，一个事务覆盖另一个事务已提交的更新数据（即A事务更新数据，然后B事务更新该数据，A事务查询发现自己更新的数据变了）。
4. **幻读**：是指当事务不是独立执行时发生的一种现象，例如第一个事务对一个表中的数据进行了修改，这种修改涉及到表中的全部数据行，同时，第二个事务也修改这个表中的数据，这种修改是向表中插入一行新数据。那么，以后就会发生操作第一个事务的用户发现表中还有没有修改的数据行，就好象发生了幻觉一样。

![image-20220414170910589](assets/image-20220414170910589.png)

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

**普通读不加锁，所以读写可以并行，读最新的数据，不管这条记录是不是已提交。不会遍历版本链，少了查找可见的版本的步骤**

## read committed (读已提交) & repeatable read (重复读)

https://wenku.baidu.com/view/a8f7e5e16237ee06eff9aef8941ea76e59fa4a4e.html



**RR级别：Innodb 通过间隙锁解决了幻读问题**

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
