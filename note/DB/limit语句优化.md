语句1：
	select * from table order by id desc  limit 150000,1000;
语句2:
	select * from table while id           limit 1000;

首先用到索引肯定能提升查找性能，其次由于limit offset,n  就算你用到索引，由于limit 与会会去扫面卡面的offset个数据，所以offset越大扫描的数据就越多。扫描越多那么回表次数也就越多，所以可以分页数据前用普通limit,后续获取到了边界值之后使用

这里的id是我们的索引列

##### id<max and limit size（不含子查询）

##### min<=id<=max

请看下面两个文章

https://zhuanlan.zhihu.com/p/29090098

https://www.cnblogs.com/weixiaotao/p/10646666.html