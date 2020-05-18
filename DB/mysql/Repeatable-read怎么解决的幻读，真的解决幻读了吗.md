不可重复读侧重于修改，幻读侧重于新增和删除。

实验准备数据：

```sql
CREATE TABLE `yunfei` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `type` int(11) DEFAULT NULL,
  `value` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `idx_type` (`type`)
) ENGINE=InnoDB AUTO_INCREMENT DEFAULT CHARSET=utf8;
 
 
INSERT INTO `yunfei` (`id`, `type`, `value`) VALUES ('1', '1', 'aa');
INSERT INTO `yunfei` (`id`, `type`, `value`) VALUES ('2', '2', 'bb');
INSERT INTO `yunfei` (`id`, `type`, `value`) VALUES ('3', '3', 'cc');
INSERT INTO `yunfei` (`id`, `type`, `value`) VALUES ('4', '4', 'dd');
INSERT INTO `yunfei` (`id`, `type`, `value`) VALUES ('5', '7', 'ee');
INSERT INTO `yunfei` (`id`, `type`, `value`) VALUES ('6', '10', 'ff');
```

实验A：

```sql
sessionA                               sessionB
 
begin;                                 begin;
 
select * from yunfei;        
id type value
1	1	aa
2	2	bb
3	3	cc
4	4	dd
5	7	ee
6	10	ff
                                        insert into yunfei(type,value) values ('9','ddddddd'); 
                                        commit;
select * from yunfei; 
id type value
1	1	aa
2	2	bb
3	3	cc
4	4	dd
5	7	ee
6	10	ff
 
insert into yunfei(type,value) values ('9','ddddddd'); 
 
[SQL]
[Err] 1062 - Duplicate entry '9' for key 'aa'
 
select * from yunfei; 
id type value
1	1	aa
2	2	bb
3	3	cc
4	4	dd
5	7	ee
6	10	ff
 
update yunfei set value='ddddddd' where type=9;
 
[SQL]
Affected rows: 0
Time: 0.000s
 
select * from yunfei; 
id type value
1	1	aa
2	2	bb
3	3	cc
4	4	dd
5	7	ee
6	10	ff
34	9	ddddddd                    
```

实验B

```sql
sessionA                                sessionB 
begin;                                 begin
 
select * from yunfei for update;        
id  type value
1	1	aa
2	2	bb
3	3	cc
4	4	dd
5	7	ee
6	10	ff
                                        insert into yunfei(type,value) values ('9','ddddddd'); 
                                        无法插入！会一直等待，直到session1 commit或rollback
 
                                        insert into yunfei(type,value) values ('11','ddddddd'); 
                                        无法插入！会一直等待，直到session1 commit或rollback
 
 
rollback;
 
begin;
 
select * from yunfei where 4<type<10 for update;   
id  type value
1	1	aa
2	2	bb
3	3	cc
4	4	dd
5	7	ee
6	10	ff
                                        insert into yunfei(type,value) values ('9','ddddddd'); 
                                        无法插入！会一直等待，直到session1 commit或rollback
 
 
                                        insert into yunfei(type,value) values ('11','ddddddd'); 
                                                       
```

结论（自己实验总结的，有错误请留言指正）：

mysql innodb只在一定程度上避免了一些幻读，但明没有真正解决幻读。

快照读（普通select）：

1：一个session永远读不到另外一个session提交的数据，避免了幻读

2：一个session在执行过程中另外一个session插入了一条记录并提交，那么在当前session重复插入的时候唯一索引冲突，明明没数据为什么冲突了？再次查询并无多数据，但是冲突提醒变相的出现了幻读。

3：一个session在执行过程中另外一个session插入了一条记录并提交，那么在当前session中修改另外一个session插入的数据提示会无数据受影响，但是再次查询多了条数据？出现幻读。

当前读（select for update）：

1·：一个session在当前读过程中没有用到索引，其他session无法插入数据，update锁了全表，避免幻读。

2：:一个session在当前读过程中用到了范围索引，那么其他session也会因为行锁（临键锁或者间隙锁）的情况无法插入，避免幻读。
————————————————

> 版权声明：本文为CSDN博主「不偷腥的mao」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
> 原文链接：https://blog.csdn.net/huiyunfei/article/details/106106550