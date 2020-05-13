# 1. 前言

[官网-Elasticsearch: 中文权威指南](https://www.elastic.co/guide/cn/elasticsearch/guide/current/getting-started.html) 这个版本里面的内容是基于2.X版本来说的，**当然推荐看下面的英文文档了，应为现在的版本，当然你看了之后可以来对照着看看中文文档里面的东西，还是有好处的**

[官网-Elasticsearch: 英文权威指南](https://www.elastic.co/guide/en/elasticsearch/reference/current/getting-started.html) 这个版本里面的内容解释了最新发布的发布版本的内容

目前最新发布的发布版本已经到了6.X

review的7.X版本也上线了

## 1.1 ES是什么

- 一个分布式的实时文档存储，*每个字段* 可以被索引与搜索,实现这个的基础是[倒排索引](Inverted-index.md)
- 一个分布式实时分析搜索引擎
- 能胜任上百个服务节点的扩展，并支持 PB 级别的结构化或者非结构化数据
- 它是建立在一个全文搜索引擎库 Apache Lucene™ (LIB)基础之上。 Lucene 可以说是当下最先进、高性能、全功能的搜索引擎库无论是开源还是私有。 

## 1.2 ES是面向文档的

https://www.elastic.co/guide/cn/elasticsearch/guide/current/_document_oriented.html

## 1.3 安装 启动

https://www.elastic.co/guide/en/elasticsearch/reference/current/getting-started-install.html

`windows`运行安装目录下的bin目录下的`elasticsearch.bat`文件就可以了

关闭的方法就是`Ctrl+C`

# 2.基础概念

了解ES中的一些核心概念有助于帮助我们更快的理解后续的内容

> ![tip](assets/tip-1549941842576.png)第二节的内容主要是做一个系统的介绍，关于更多的ES集群相关内容请查看[集群内原理](cluster-guide.md)

## 2.1 Near realtime(NRT)

Elasticsearch  是一个接近实时的搜索平台，这意味着从索引文档到可搜索文档的时间有一点延迟（通常为一秒）。

## 2.2 Cluster

集群是一个或多个节点（服务器）的集合，它们共同保存您的整个数据，并提供跨所有节点的联合索引和搜索功能。 集群由唯一名称标识，默认情况下为 `“elasticsearch”`。 名称很重要，因为节点是按名称加入群集的，相同名称的节点自动加入到一个集群中

确保不要在不同的环境中重用相同的群集名称，否则最终会导致节点加入错误的集群。 例如，您可以将logging-dev，logging-stage和logging-prod用于开发，登台和生产集群。

请注意，如果群集中只有一个节点，那么它是完全正常的。 此外，您还可以拥有多个独立的集群，每个集群都有自己唯一的集群名称。

**修改cluster name**

你可以在 `elasticsearch.yml` 配置文件中 修改 `cluster.name`这个文件会在`elasticsearch`启动的时候加载,这个文件就存在于你的 elasticsearch 安装目录之中的config目录之中

## 2.3 Node

节点是单个服务器，并作为集群的一部分,存储数据并参与集群的索引和搜索功能。 就像集群一样，节点由名称标识，**默认情况下，该名称是在启动时分配给节点的随机通用唯一标识符（UUID）。 如果不需要默认值，可以定义所需的任何节点名称。 此名称对于管理目的非常重要，您可以在其中识别网络中哪些服务器与Elasticsearch集群中的哪些节点相对应。**

**可以将节点配置为按群集名称加入特定群集**。 默认情况下，每个节点都设置为加入名为`elasticsearch`的集群，这意味着如果您在网络上启动了多个节点并且假设它们可以相互发现 - 它们将自动形成并加入一个名为`elasticsearch`的群集。

在单个群集中，您可以拥有任意数量的节点。 此外，如果您的网络上当前没有其他Elasticsearch节点正在运行，则默认情况下启动单个节点将形成一个名为`elasticsearch`的新单节点集群。

## 2.4 Index

索引是具有某些类似特征的文档集合。 例如，您可以拥有客户数据的索引，产品目录的另一个索引以及订单数据的另一个索引。 索引由名称标识（必须全部小写），此名称用于在对其中的文档执行索引，搜索，更新和删除操作时引用索引。

在单个集群中，您可以根据需要定义任意数量的索引。

**index 相当于 SQL概念中的 数据库**

## 2.5 Type

type曾经是index中的类别却分，es允许在同一个index中存储不同type的文档，例如，一个index中，一种用于存用户信息，一种存博客帖子。在今后将慢慢的删除这个概念，也就是说要将`1对n的关系改成1对1`。移除的原因：[*Removal of mapping types*](https://www.elastic.co/guide/en/elasticsearch/reference/current/removal-of-types.html) 

> ![tip](assets/tip-1549941842576.png)在Elasticsearch 6.0.0或更高版本中创建的索引可能只包含单个映射类型。 在具有多种映射类型的5.x中创建的索引将继续像以前一样在Elasticsearch 6.x中运行。 类型将在Elasticsearch 7.0.0中的API中弃用，并在8.0.0中完全删除。
>

**type 相当于 SQL概念中的 表**

## 2.6 Document

文档是可以编制索引的基本信息单元。 例如，您可以为单个客户提供文档，为单个产品提供另一个文档，为单个订单提供另一个文档。 该文档以`JSON`（`JavaScript Object Notation`）表示，它是一种普遍存在的互联网数据交换格式。

在索引/类型中，您可以根据需要存储任意数量的文档。 请注意，尽管文档实际上位于索引中，但实际上必须将文档编入索引/分配给索引中的类型。

**ducument 相当于 SQL概念中的 记录**

## 2.7 Shards & Replicas

**默认情况下，在创建索引的时候会给他设置5个shard,1个replicas**

### Shards

由于一个索引可能存储的数据量会很大，可能超过单个节点的硬件的数据量阈值，也可能到达一定量，可能导致对单个节点的搜索会变得缓慢

为了解决这个问题，elasticsearch 提供了将索引细分为多个的功能，这就是分片，创建索引是可以指定所需分片的数量，每个分片本身是一个功能齐全且独立的“索引”，可以托管在集群中的任何节点上。

分片很重要，主要有两个原因：

* 它允许你做水平扩展/收缩
* 它允许您跨分片,分布数据以及并行化操作，从而提高性能/吞吐量

分片的分布方式以及如何将文档聚合回搜索请求的机制完全由Elasticsearch管理

### Replicas

在网络/云环境中任何时候都可能出现故障，所以elasticsearch中加入了一种故障转移机制，以保证系统的高可用，这就是副本，Elasticsearch允许您将索引的分片的一个或多个副本制作成所谓的副本分片或简称副本。

副本很重要，主要有两个原因：

* 它在shard/node出现故障时提供高可用性。 因此，**请务必注意，副本分片永远不会在与从中复制的原始/主分片在相同的节点上分配。**
* 它允许您扩展搜索量/吞吐量，因为可以在所有副本上并行执行搜索。

可以在创建索引时为每个索引定义分片和副本的数量。 

* 创建索引后，您还可以随时动态更改副本数。 

* 您可以使用_shrink和_split API更改现有索引的分片数，但这不是一项简单的任务，预先计划正确数量的分片是最佳方法。

**默认情况下，Elasticsearch中的每个索引都分配了5个主分片和1个副本，这意味着如果群集中至少有两个节点，则索引将包含5个主分片和另外5个副本分片（1个完整副本），总计为 每个索引10个分片。**



# 3. REST API

## 3.1 简介

如果你启动并运行了节点（和集群）之后，下一步是了解如何与它进行通信。Elasticsearch提供了一个非常全面和强大的REST API，您可以使用它与集群进行交互。 使用API可以完成的一些事项如下：

- Check your cluster, node, and index health, status, and statistics

  检查群集，节点和索引运行状况，状态和统计信息

- Administer your cluster, node, and index data and metadata

  管理您的群集，节点和索引数据和元数据

- Perform CRUD (Create, Read, Update, and Delete) and search operations against your indexes

  对索引执行CRUD（创建，读取，更新和删除）和搜索操作

- Execute advanced search operations such as paging, sorting, filtering, scripting, aggregations, and many others

  执行高级搜索操作，例如分页，排序，过滤，脚本编写，聚合等等

## 3.2 REST API pattern 

```
<HTTP Verb> /<Index>/<Type>/<ID> [source]
```

很好理解的东西：

* http verb : http协议的动词，例如 get,put,post .....
* index : elsticsearch 的索引名
* type: elsticsearch 的索引的类型（在[2.5节](#2.5 Type)中我们说过这个东西将在之后的版本中废弃）
* id: 相当于 `key` **在es种这个叫做`id**

例子：

```json
PUT /megacorp/employee/1 
{
    "first_name" : "John",
    "last_name" :  "Smith",
    "age" :        25,
    "about" :      "I love to go rock climbing",
    "interests": [ "sports", "music" ]
}
```

> ![tip](assets/tip-1549941842576.png)上面的请求原版是 PUT /megacorp/employee/1?pretty 样子的，但是由于?pretty在指令中默认的，所以在后面使用到这个的时候我都会省略掉

上面的请求包含了5部分的信息，这里介绍前文中没有说明的：

* `PUT` 

  http verb:http协议的动词

- `megacorp`

  index ,索引名称

- `employee`

  type,类型名称

- `1`

  id，也就是相当于 `key` 

- 后面的json信息，也就是我们的docment数据了 相当于`value`

  **在es种这个叫做`source`**

## 3.3 获得集群中的节点列表

我们会用到  [`_cat` API](https://www.elastic.co/guide/en/elasticsearch/reference/6.6/cat.html)

```
GET /_cat/nodes?v
```

响应数据

```
ip        heap.percent ram.percent cpu load_1m load_5m load_15m node.role  
127.0.0.1           43          54  16                          mdi       

master   name
*        ZA1VBTg
```

在这里，我们可以看到一个名为“ZA1VBTg”的节点，它是我们集群中当前的单个节点


## 3.4 获取索引列表

我们会用到  [`_cat` API](https://www.elastic.co/guide/en/elasticsearch/reference/6.6/cat.html)

```js
GET /_cat/indices?v
```

响应数据

```
health status index     uuid                   pri rep docs.count 

docs.deleted store.size pri.store.size
          
```

现在没有任何索引

> ![tip](assets/tip-1549941842576.png)上文 3.3 、 3.4 章节我们可以发现，都用到了 [`_cat` API](https://www.elastic.co/guide/en/elasticsearch/reference/6.6/cat.html)，从中我们也能知道，es的集群，集群中的节点，分片，副本，索引......的状态信息都存储在`_cat`当中

## 3.5 索引

### 创建

接下来我们创建一个 `customer`的索引，然后再次获取索引列表

```js
PUT /customer
GET /_cat/indices?v
```

可以看到这样的响应数据

```
health status index    uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   customer 95SQ4TSUT7mWBT7VNHH67A   5   1          0            0 260b           260b

```

这个信息告诉我们，这里有一个叫做`customer`的索引，他有5个primary的shards 和 一个`replica`（[前面的章节](#2.7 Shards & Replicas)中我提到过这个值是默认的）

document数量为0，被删除的document数量也为0，占用内存260bit

为什么这个索引在存在shards+replica的情况下会是yellow状态呢？因为我这里用的是一台机器在做演示，也就是集群中只有一个节点，这个副本和shards分配在一个节点上，这个不满足高可用性，所以状态是yellow

**在插入数据的同时也是可以创建索引的**，[更多内容](#更多 API )

> ![tip](assets/tip-1549941842576.png)如何再创建index的时候指定shards & replica呢？使用下面的方式就可以做到了
>
> ```json
> PUT /blogs
> {
>    "settings" : {
>       "number_of_shards" : 3,
>       "number_of_replicas" : 1
>    }
> }
> ```

### 删除

```js
DELETE /megacorp
```

可以看到响应了我们

```json
{
    'acknowledged':true
}
```

## 3.7 docment的增删改查

### 3.7.1 插入

可以使用 put/post 进行 insert/replace

```json
PUT /megacorp/employee/1
{
    "first_name" : "John",
    "last_name" :  "Smith",
    "age" :        25,
    "about" :      "I love to go rock climbing",
    "interests": [ "sports", "music" ]
}
```

如果 /megacorp/employee/1 的数据不存在，那么就会去进行插入操作，如果存在那么就会替换这份数据

```json
PUT /megacorp/employee
{
    "first_name" : "John",
    "last_name" :  "Smith",
    "age" :        25,
    "about" :      "I love to go rock climbing",
    "interests": [ "sports", "music" ]
}
```

> ![tip](assets/tip-1549941842576.png)但是不推荐这样使用，因为这样插入进去的数据，id不是你控制的

### 3.7.2 删除

```
DELETE /megacorp/employee/1
```

执行这个条命令之后的响应是这样的

```json
{
  "_index" : "megacorp",
  "_type" : "employee",
  "_id" : "4",
  "_version" : 2,
  "result" : "deleted",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 9,
  "_primary_term" : 1
}
```

这样这条数据就会被删除了，"result" : "deleted",说明这文档存在并且被删除了

如果"result" : "not found",证明这个文档不存在

> ![tip](assets/tip-1549941842576.png)当然es也提供了[`_delete_by_query` API](delete_by_query-API) （类似 SQL DELETE-WHERE语句的功能）

### 3.7.3 更新

我们使用 POST 来对索引中的对应类型的文档进行修改

#### 普通更新

这里需要注意的是`_update`，以及json内容中的`doc`

```json
POST /megacorp/employee/1/_update
{
  "doc": {
    "last_name" :  "J",
    "age" : 25,
    "about" : "I like to listening music",
    "interests" : [
      "music"
    ]
  }
}
```

然后我们，再次查询一个数据

```
GET /megacorp/employee/1
```

可以看到**数据被我们修改了一部分**

```json
{
  "_index" : "megacorp",
  "_type" : "employee",
  "_id" : "6",
  "_version" : 4,
  "found" : true,
  "_source" : {
    "first_name" : "John",
    "last_name" : "J",
    "age" : 25,
    "about" : "I like to listening music",
    "interests" : [
      "music"
    ]
  }
}
```

> ![tip](assets/tip-1549941842576.png)修改的时候，我们还可以加入新的数据，比如把，source部分换成
>
> ```
> {
>   	"doc": {
>     		"last_name" :  "J",
> 		"sex":"male"
>        }
> }
> ```
>
> 这样就再原文档中添加了，"sex":"male"数据

#### 带script 脚本操作的更新

```
POST /megacorp/employee/1/_update
{
  "script" : "ctx._source.age += 5"
}
```

这样，文档中的age的数据就会增加了

> ![tip](assets/tip-1549941842576.png)这个其实就是 类似js的操作，所以同样不同类型会有区别，比如这个的age字段是数字，所以+5之后变成30，如果是一个字符，那么就会编程再字符后面+5，也是age：“25” 最后的结果就是 age:"255"



> ![tip](assets/tip-1549941842576.png)**Elasticsearch提供了在给定查询条件（如SQL UPDATE-WHERE语句）的情况下更新多个文档的功能。**[`docs-update-by-query` API](https://www.elastic.co/guide/en/elasticsearch/reference/6.6/docs-update-by-query.html) 

### 3.7.2 查询

#### 普通查询

```js
GET /megacorp/employee/1
```

response

```json
{
  "_index" : "megacorp",
  "_type" : "employee",
  "_id" : "1",
  "_version" : 1,
  "found" : true,
  "_source" : {
    "first_name" : "John",
    "last_name" : "Smith",
    "age" : 25,
    "about" : "I love to go rock climbing",
    "interests" : [
      "sports",
      "music"
    ]
  }
}
```

#### 查询所有: 

例如`GET /megacorp/employee/_search`

#### 查询带有特定值的文档

`GET /megacorp/employee/_search?q=last_name:Smith`

#### 查询表达式

有了查询表达式，我们可以构建更加复杂和健壮的查询

比如我们现在重写4.2.2 的查询

```json
GET /megacorp/employee/_search
{
    "query" : {
        "match" : {
            "last_name" : "Smith"
        }
    }
}
```

返回结果与之前的查询一样，但还是可以看到有一些变化。其中之一是，不再使用 *query-string* 参数，而是一个请求体替代。这个请求使用 JSON 构造，并使用了一个 `match` 查询（属于查询类型之一，后续将会了解）。

## 3.8 批处理

elasticsearch处理提供了对于索引，文档的基础增删改查操作之外，还提供了增对他们的批处理方式—— [`_bulk` API](https://www.elastic.co/guide/en/elasticsearch/reference/6.6/docs-bulk.html).**该功能非常重要，因为它提供了一种有效的机制尽可能快的进行多个操作，同时减少了网络的IO次数**

下面我们来看一个简单的例子，在一个批量操作中索引两个文档

```json
POST /megacorp/employee/_bulk
{"index":{"_id":"2"}}
{"first_name": "Douglas" }
{"index":{"_id":"6"}}
{"first_name": "ALan" }
```

## 更多 API 

[Document-APIS](Document-APIS.md)

## 更多案例

https://www.elastic.co/guide/cn/elasticsearch/guide/current/_finding_your_feet.html

# 4. 程序语言和Elasticsearch交互

### 3.1 java API

使用java client 代码中可以使用elasticsearch内置的两个客户端：

#### 3.1.1 node client

`node client` 作为一个非数据加入到本地集群中，换句话说，它本身不保存任何数据，但是它知道数据在集群中的哪个节点中，并且可以把请求转发到正确的节点。

#### 3.1.2 Transport client

轻量级的传输客户端，可以将请求发送到远程集群中的一个节点上。

两个 Java 客户端都是通过 *9300* 端口并使用 Elasticsearch 的原生传输协议和集群交互。集群中的节点通过端口 9300 彼此通信。如果这个端口没有打开，节点将无法形成一个集群。

> ![tip](assets/tip-1549941842576.png)java 客户端作为节点必须和 Elasticsearch 有相同的 *主要* 版本；否则，它们之间将无法互相理解。

### 3.2 更多

https://www.elastic.co/guide/en/elasticsearch/client/index.html

# 5. 可视化工具—Kibana

## 简介

Kibana在es生态种是一个强大的可视化工具，更多的内容请自行阅读[官网](https://www.elastic.co/products/kibana)

**基本使用**

下载安装：https://www.elastic.co/products/kibana

启动：

* linux: `./bin/kibana `
* windows:`bin\kibana.bat`

> ![tip](assets/tip-1549941842576.png)启动Kibana之前要先启动elasticserch哦

浏览器进入控制页面: http://localhost:5601/app/kibana#/dev_tools

在这里提供了小工具，可以基于一些列的 restful 指令和 elasticsearch进行交互



