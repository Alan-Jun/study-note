# 5.1 XA协议

首先我们来简要看下分布式事务处理的XA规范

![img](https:////upload-images.jianshu.io/upload_images/5879294-867d547beb98c600.png?imageMogr2/auto-orient/strip|imageView2/2/w/863/format/webp)

可知XA规范中分布式事务有AP，RM，TM组成：

- 其中应用程序(Application Program ，简称AP)：AP定义事务边界（定义事务开始和结束）并访问事务边界内的资源。

- 资源管理器(Resource Manager，简称RM)：Rm管理计算机共享的资源，许多软件都可以去访问这些资源，资源包含比如数据库、文件系统、打印机服务器等。
- 事务管理器(Transaction Manager ，简称TM)：负责管理全局事务，分配事务唯一标识，监控事务的执行进度，并负责事务的提交、回滚、失败恢复等。

Xa主要规定了RM与TM之间的交互，下面来看下XA规范中定义的RM 和 TM交互的接口：

![img](https:////upload-images.jianshu.io/upload_images/5879294-c13511a063f340e9.png?imageMogr2/auto-orient/strip|imageView2/2/w/887/format/webp)

本图来着 参考文章XA规范25页

- xa_start负责开启或者恢复一个事务分支，并且管理XID到调用线程
- xa_end 负责取消当前线程与事务分支的关联
- xa_prepare负责询问RM 是否准备好了提交事务分支
- xa_commit通知RM提交事务分支
- xa_rollback  通知RM回滚事务分支

XA协议是使用了二阶段协议的，其中：

- 第一阶段TM要求所有的RM准备提交对应的事务分支，询问RM是否有能力保证成功的提交事务分支，RM根据自己的情况，如果判断自己进行的工作可以被提交，那就就对工作内容进行持久化，并给TM回执OK；否者给TM的回执NO。RM在发送了否定答复并回滚了已经的工作后，就可以丢弃这个事务分支信息了。
- 第二阶段TM根据阶段1各个RM prepare的结果，决定是提交还是回滚事务。如果所有的RM都prepare成功，那么TM通知所有的RM进行提交；如果有RM prepare回执NO的话，则TM通知所有RM回滚自己的事务分支。

也就是TM与RM之间是通过两阶段提交协议进行交互的。

## 5.2 MySQL中XA实现

MYSQL的数据库存储引擎InnoDB的事务特性能够保证在存储引擎级别实现ACID，而分布式事务让存储引擎级别的事务扩展到数据库层面，甚至扩展到多个数据库之间，这是通过两阶段提交协议来实现的，MySQL 5.0或者更新版本开始支持XA事务，从下图可知MySQL中只有InnoDB引擎支持XA协议：

![img](https:////upload-images.jianshu.io/upload_images/5879294-cce9cbb77cb1e13b.png?imageMogr2/auto-orient/strip|imageView2/2/w/954/format/webp)

Mysql中存在两种XA事务，一种是内部XA事务主要用来协调存储引擎和二进制日志，一种是外部事务可以参与到外部分布式事务中（比如多个数据库实现的分布式事务），本节我们主要讨论外部事务。

在MySQL数据库分布式事务中，MySQL是XA事务过程中的资源管理器（RM）存在的，TM是连接MySQL服务器的客户端。MySQL数据库是作为RM存在的，在分布式事务中一般会涉及到至少两个RM，所以我们说的MySQL支持XA协议是说mysql作为RM来说的，也就是说MySQL实现了XA协议中RM应该具有的功能；需要注意的是MySQL中只有当隔离级别为Serializable时候才能使用分布式事务，所以需要使用`set global tx_isolation='serializable',session tx_isolation='serializable';`设置数据库隔离级别（具体可以参考[本地事务](https://gitbook.cn/gitchat/activity/5b339cc2b3d1de6cd5e3cecb)）。

下面我们来看看在MySQL数据库单个节点运行XA事务，首先来看下MySQL下xa事务语法：

![img](https:////upload-images.jianshu.io/upload_images/5879294-279e620c6770db6f.png?imageMogr2/auto-orient/strip|imageView2/2/w/804/format/webp)

其中xid是一个全局唯一的id标示一个分支事务，每个分支事务有自己的全局唯一的一个id,是一个字符串。
 然后确认下mysql是否启动了xa功能：



![img](https:////upload-images.jianshu.io/upload_images/5879294-4a5e660fb3a4c59a.png?imageMogr2/auto-orient/strip|imageView2/2/w/958/format/webp)
 可知启动了，下面具体看一个实例：

![img](https:////upload-images.jianshu.io/upload_images/5879294-efb03c20431c8965.png?imageMogr2/auto-orient/strip|imageView2/2/w/955/format/webp)

- 其中首先使用XA START ‘xid' 启动了一个XA事务，并把它置于ACTIVE状态
- 对于一个ACTIVE状态的 XA事务，我们可以执行构成事务的多条SQL语句，也就是指定分支事务的边界，然后执行一个XA END ‘xid'语句，XA END把事务放入IDLE状态，也就是结束事务边界，在xa start和xa end之间的语句就构成了本分支事务的一个事务范围。当调用xa end 'xid1'后由于结束了事务边界，所以这时候如何在执行sql语句会抛出ERROR 1399 (XAE07): XAER_RMFAIL: The command cannot be executed when global transaction is in the  IDLE state错误，也就是当分支事务处于IDLE状态时候不允许执行没有包含到分支事务边界里面的其他sql.

- 对于一个IDLE 状态XA事务，可以执行一个XA PREPARE语句或一个XA COMMIT…ONE PHASE语句,其中XA PREPARE把事务放入PREPARED状态。在此点上的XA RECOVER语句将在其输出中包括事务的xid值，因为XA RECOVER会列出处于PREPARED状态的所有XA事务。XA COMMIT…ONE PHASE用于预备和提交事务，也就是转换为一阶段协议，直接提交事务。
- 对于一个PREPARED状态的 XA事务,可以执行XA COMMIT 语句来提交或者执行XA ROLLBACK来回滚xa事务。

其中二阶段协议中第一阶段是执行 xa prepare时候，这时候MySQL客户端（TM）向MySQL数据库服务器（RM）发出prepare"准备提交"请求，数据库收到请求后执行数据修改和日志记录等处理，处理完成后只是把事务的状态改成"可以提交",然后把结果返回给事务管理器。

如果第一阶段中数据库都prepare成功，那么mysql客户端（TM）向数据库服务器发出"commit"请求，数据库服务器把事务的"可以提交"状态改为"提交完成"状态，然后返回应答。如果在第一阶段内数据库的操作发生了错误，或者mysql客户端（RM）收不到数据库的回应，则认为事务失败，执行rollback回撤所有数据库的事务。

上面例子是在一个数据库节点上运行的一个分支事务，演示了单个数据库上执行xa分支事务的流程，但是通常都是使用编程语言，比如Java的 JTA来完成MySQL的分布式事务的，下面一个例子用来演示：
 首先添加依赖

```java
    <dependency>
            <groupId>javax.transaction</groupId>
            <artifactId>jta</artifactId>
            <version>1.1</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>6.0.6</version>
        </dependency>
```

代码：

```java
public class XaDemo {

    public static MysqlXADataSource getDataSource(String connStr, String user, String pwd) {

        try {

            MysqlXADataSource ds = new MysqlXADataSource();
            ds.setUrl(connStr);
            ds.setUser(user);
            ds.setPassword(pwd);

            return ds;
        } catch (Exception e) {
            e.printStackTrace();
        }

        return null;
    }

    public static void main(String[] arg) {
        String connStr1 = "jdbc:mysql://192.168.0.1:3306/test";
        String connStr2 = "jdbc:mysql://192.168.0.2:3306/test";

        try {
            //从不同数据库获取数据库数据源
            MysqlXADataSource ds1 = getDataSource(connStr1, "root", "123456");
            MysqlXADataSource ds2 = getDataSource(connStr2, "root", "123456");

            //数据库1获取连接
            XAConnection xaConnection1 = ds1.getXAConnection();
            XAResource xaResource1 = xaConnection1.getXAResource();
            Connection connection1 = xaConnection1.getConnection();
            Statement statement1 = connection1.createStatement();
            
            //数据库2获取连接
            XAConnection xaConnection2 = ds2.getXAConnection();
            XAResource xaResource2 = xaConnection2.getXAResource();
            Connection connection2 = xaConnection2.getConnection();
            Statement statement2 = connection2.createStatement();

            //创建事务分支的xid
            Xid xid1 = new MysqlXid(new byte[] { 0x01 }, new byte[] { 0x02 }, 100);
            Xid xid2 = new MysqlXid(new byte[] { 0x011 }, new byte[] { 0x012 }, 100);

            try {
                //事务分支1关联分支事务sql语句
                xaResource1.start(xid1, XAResource.TMNOFLAGS);
                int update1Result = statement1.executeUpdate("update account_from set money=money - 50 where id=1");
                xaResource1.end(xid1, XAResource.TMSUCCESS);

                //事务分支2关联分支事务sql语句
                xaResource2.start(xid2, XAResource.TMNOFLAGS);
                int update2Result = statement2.executeUpdate("update account_to set money= money + 50 where id=1");
                xaResource2.end(xid2, XAResource.TMSUCCESS);
                
                // 两阶段提交协议第一阶段
                int ret1 = xaResource1.prepare(xid1);
                int ret2 = xaResource2.prepare(xid2);

                // 两阶段提交协议第二阶段
                if (XAResource.XA_OK == ret1 && XAResource.XA_OK == ret2) {
                    xaResource1.commit(xid1, false);
                    xaResource2.commit(xid2, false);

                    System.out.println("reslut1:" + update1Result + ", result2:" + update2Result);
                }
            } catch (Exception e) {
                e.printStackTrace();
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

}
```

如上代码对两个机器上的数据库进行转账操作。

作者：阿里加多
链接：https://www.jianshu.com/p/a7d1c4f2542c
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。