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

  `set session transaction isolatin level repeatable read;`

* 设置系统当前隔离级别

  `set global transaction isolation level repeatable read;`

# 四个隔离界别如何实现的

## Read uncommitted(读未提交) 

查询引擎去按照条件取数据 ，数据可能在内存中，也可能还在磁盘中，取到数据之后，作为当前数据 ，不管是否提交，不管重复读是否数据不一致）

## Read committed (读已提交)

RC级别: read view 会在每一个 select语句执行的时候都会去重新拉一次数据，这决意味着每一次查询数据使用用来判断得read view都是最新的，保证读取到的一定是提交之后的数据

1. 查询引擎去按照条件取数据 ，数据可能在内存中，也可能还在磁盘中，取到数据之后作为当前数据
2. 然后使用该数据中存放的事务id ,将其作为[read_view_sees_trx_id函数](#read_view_sees_trx_id) 的trx_id参数，判断是否该数据是否可用，
3. 可用直接返回,这时候读取到的一定是提交了的数据（可能是最新的，也可能是undo log的）
4. 如果不可用，通过undo log指针找到上一个版本的数据，然后作为当前数据，回到第二部继续判断

## Repeatable read (重复读)

**RR级别：read view 数据是在事务第一次读开始只创建一次，这样可以保证之后的重复读的 read view 相同，也就能保证[read_view_sees_trx_id函数](#read_view_sees_trx_id) 的判断一致，查询到的数据自然就一致了**

## Serializable 界别

运用锁机制来达到完全同步

> 更多关于MVCC的请看[MVCC](MVCC.md)

## read_view_sees_trx_id

read view 中存放了当前未commit得活跃的事务id列表
`[up_limit_id，low_limit_id ]`

**1. 当行记录的事务ID小于当前的最小活动id，就是可见的。**
　　if (trx_id < view->up_limit_id) {
　　　　return(TRUE);
　　}
**2. 当行记录的事务ID大于当前的最大活动id，就是不可见的。**
　　if (trx_id >= view->low_limit_id) {
　　　　return(FALSE);
　　}
**3. 行记录的事务ID在活动范围之中时，判断是否在活动链表中，如果在那就是未commit就不可见，如果不在就是可见的。**
　　for (i = 0; i < n_ids; i++) {
　　　 view_trx_id
　　　　　　= read_view_get_nth_trx_id(view, n_ids - i - 1);
　　　　if (trx_id <= view_trx_id) {
　　　　return(trx_id != view_trx_id);
　　　　}
　　}

****