# 简介

ZK 是CP系统 ：写强一致性，读顺序一致性 https://blog.csdn.net/weixin_43854800/article/details/127983835

ZooKeeper: A Distributed Coordination Service for Distributed Applications

ZooKeeper主要**服务于分布式系统**，可以用ZooKeeper来做：统一配置管理、统一命名服务、分布式锁、集群管理。

使用分布式系统就无法避免对节点管理的问题(需要实时感知节点的状态、对节点进行统一管理等等)，而由于这些问题处理起来可能相对麻烦和提高了系统的复杂性，ZooKeeper作为一个能够**通用**解决这些问题的中间件就应运而生了。

> https://mp.weixin.qq.com/s/J2is0iapoxn5ZuQySX9SKw

Zookeeper遵循是的CP还是AP原则，这个挺难的，因为它并不是一个完整的强一致性的协议实现，然后它在选举的时候会暂停对外服务，这个又不满足可用性，不过如果从半数同步的这点来看姑且算是满足一致性，我认为会偏向 CP 一些，不过会有一个更好的解释那就是BASE (允许数据同步存在时延，数据是最终一致的，选举期间访问直接快速失败，client有local cache 算是基本可用)



Eureka遵循的是AP原则，即保证了高可用，失去了一执行。每台服务器之间都有心跳检测机制，而且每台服务器都能进行读写，通过心跳机制完成数据共享同步，所以当一台机器宕机之后，其他的机器可以正常工作，但是可能此时宕机的机器还没有进行数据共享同步，所以其不满足一致性。

> CAP理论指的是在一个分布式系统中，不可能同时满足Consistency（一致性）、Availablity（可用性）、Partition tolerance（分区容错性）这三个基本需求，最多只能满足其中的两项。
>
> 一致性（Consistency）：数据在不同的副本之间数据是保持一致的，并且当执行数据更新之后，各个副本之间能然是处于一致的状态。
>
> 可用性（Availablity）：系统提供的服务必须是处于一直可用的状态，针对每一次对系统的请求操作在设定的时间内，都能得到正常的result返回。
>
> 分区容错性（Partition tolerance）：分布式系统在遇到任何网络分区故障的时候，仍然需要能够保证对外提供满足一致性和可用性的服务，除非整个网络环境全部瘫痪了。

# [Paxos协议](../分布式/一致性协议.md)

# [ZAB 协议](../分布式/一致性协议.md)

# 脑裂问题

https://wenku.baidu.com/view/a33e4838a46e58fafab069dc5022aaea998f41bd.html

## 安装部署

集群部署

单点部署

伪集群部署

参考——《从Paxos到Zookeeper  分布式一致性原理与实践 [倪超著]》第五章的说明

 

## Zoo.cfg 文件常见配置——更多配置去官网查询

tickTime=2000

 

initLimit=10

 

syncLimit=5

 

dataDir=/home/ymj/datadir/zookeeper

 

clientPort=2181（客户端的端口）

 

\#maxClientCnxns=60

 

\# 在打开autopurge之前，请务必阅读管理员指南的维护部分。 

\#

\# 链接：http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance

\#

 

\#autopurge.snapRetainCount=3

 

\#autopurge.purgeInterval=1

 

server.1=192.168.93.128:2888:3888

server.2=192.168.93.129:2888:3888

server.3=192.168.93.130:2888:3888

 

 

下面解释一下 集群部署的这个值

server.id=ip:port1:port2

id 服务器id

ip 服务器ip

port1 集群的机器间通信（数据同步）的端口

port2 集群间的选举的端口

 

## 常见异常

### 端口被占用

![img](assets\clip_image002.jpg)

 

### 磁盘没有剩余空间

![img](assets\clip_image004.jpg)

![img](assets\clip_image006.jpg)

 

### 无法找到myid 文件

![img](assets\clip_image008.jpg)

 

### 集群中其他机器的leader 选举端口未打开

![img](assets\clip_image010.jpg)

![img](assets\clip_image012.jpg)

 

## 可执行脚本

![img](assets\clip_image014.jpg)

### zkServer.sh

sh zkServer.sh   start/stop/status

 

### 客户端脚本zkCli.sh

运行之后 会进入到简易客户端模式

sh zkCli.sh （连接本地的zookeeper） 

sh zkCli.sh -server ip:port 连接对应的服务器的 zookeeper

 

 

 

进入客户端模式之后 使用 help 命令就可以查看各种指令了

![img](assets\clip_image016.jpg)

 

#### 创建节点 (增) create

create [-s] [-e] path data acl

 

create -s -e path data acl创建有序临时节点

 

create -s path data acl创建有序节点

![img](assets\clip_image018.jpg)

-e 创建临时检点

![img](assets\clip_image020.jpg)

 

什么都不写 创建普通的持久节点

![img](assets\clip_image022.jpg)

#### 读取节点（查）ls + get

ls 和linux 的ls 命令类似

![img](assets\clip_image024.jpg)

get 类似于  cat 命令 可以查看节点里面的文本内容

 

临时节点：

![img](assets\clip_image026.jpg)

 

持久节点：

![img](assets\clip_image028.jpg)

 

 

#### 节点内容

cZxid = 0x30000000f   //创建节点时的事务id

ctime = Sun Jun 10 18:36:18 CST 2018 //创建的时间

mZxid = 0x30000000f //最近一次修改节点时候的 事务id m=modify

mtime = Sun Jun 10 18:36:18 CST 2018 //最近一次修改节点时候的时间

pZxid = 0x30000000f   //子节点列表最后一次被修改的事务id

cversion = 0  //节点版本号

dataVersion = 0 //数据版本号 会随着数据的修改而改变

aclVersion = 0 //权限版本

ephemeralOwner = 0x0 //创建该临时节点的会话sessionid ,如果这是一个持久节点那值为0x0

dataLength = 10 //数据的字符长度

numChildren = 0 // 子节点的数量

#### 更新（改）set

set path data [version]

version 用于指定本次更新是基于哪一个数据版本（dataVersion）

如果此时版本不匹配  是无法修改的

#### 删除（删）delete

delete path [version]

需要注意的是 要删除一个节点必须保证他没有子节点，如果穿在子节点，需要将子节点全部删除，在删除目标节点。

### Zookeeper的Java 客户端api 使用：



# zookeeper读写请求方式

https://www.csdn.net/tags/NtzaMg1sNzAyMTctYmxvZwO0O0OO0O0O.html

写请求交给 leader 是因为 zookeeper的分布式一致性协议决定的

# Zookeeper 的脑裂问题

https://blog.csdn.net/zxylwj/article/details/103608916

