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

