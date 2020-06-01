# Index API

索引不存在的情况下，我们可以有两种方式创建索引，这里先演示第一种

```
PUT /person
```

执行之后可以看到这样的响应

```json
{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "person"
}
```

这就说明名称为person的索引创建成功了

通过下面这条命令也能看到，我们整个elasticserch中存在的索引

request

```
GET /_cat/indices?v
```

response

```
health status index     uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   .kibana_1 z9nkdnbaTg23l-pNTBuBrg   1   0          3            0 11.9kb     11.9kb    
yellow open   person    1AwHtVdaT2SfKlPOZa4WyQ   5   1          0            0 1.1kb      1.1kb     

```

我们能很明显的看到有一个名为 person的索引，证明索引创建成功了

## 自动创建索引

首先我们先看一个例子

```json
PUT twitter/_doc/1
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
```

`twitter` 索引一开始并不存在，我这时候直接向这个索引中插入数据，得到的响应是

```json
{
  "_index" : "twitter",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 0,
  "_primary_term" : 1
}
```

从响应数据`"_shards"`中我们看到这个操作成功了，为什么能成功了，这自然是elasticseach给我们自动创建了`twitter` 这个索引

**自动创建索引由`action.auto_create_index`设置控制。 此设置默认为true，表示始终自动创建索引。通过将此设置的值更改为特定的值，我们可以对这个操作进行控制，目前有两种情况**

- 指定该字段为一个true/false,

  - true：启用索引的自动创建
  - false：禁止索引的自动创建

  例：

  ```json
  PUT _cluster/settings
  {
      "persistent": {
          "action.auto_create_index": true
      }
  }
  ```

- 指定能创建的索引名称列表，仅允许对匹配列表中特定模式的索引自动创建索引。 它也可以通过在列表中用+或 - 的前缀来明确允许和禁止。例子：

  request

  ```json
  PUT _cluster/settings
  {
      "persistent": {
          "action.auto_create_index": "twitter,index10,-stunden*t" 
      }
  }
  ```

  response

  ```json
  {
    "acknowledged" : true,
    "persistent" : {
      "action" : {
        "auto_create_index" : "twitter,index10,-stunden*t"
      }
    },
    "transient" : { }
  }
  
  ```

  响应数据告诉我们，修改`action.auto_create_index` 状态成功，这时候我们在来试一下自动创建

  request

  ```json
  PUT student/_doc/1
  {
      "user" : "kimchy",
      "post_date" : "2009-11-15T14:12:12",
      "message" : "trying out Elasticsearch"
  }
  ```

  很明显我们的`student`索引并不存在，按正常情况，这次创建是能够成功的，但是我们指定了

  `"auto_create_index" : "twitter,index10,-student*"`, 这样的规则，也就是任何名为student*的索引都不能自动创建

  让我们看一下响应结果

  ```json
  {
    "error" : {
      "root_cause" : [
        {
          "type" : "index_not_found_exception",
          "reason" : "no such index and [action.auto_create_index] contains [-student*] which forbids automatic creation of the index",
          "index_uuid" : "_na_",
          "index" : "student"
        }
      ],
      "type" : "index_not_found_exception",
      "reason" : "no such index and [action.auto_create_index] contains [-student*] which forbids automatic creation of the index",
      "index_uuid" : "_na_",
      "index" : "student"
    },
    "status" : 404
  }
  
  ```

  错误原因response中写的很清楚。

  我们在试一下规则中允许的`index10`

  request

  ```json
  PUT index10/_doc/1
  {
      "user" : "kimchy",
      "post_date" : "2009-11-15T14:12:12",
      "message" : "trying out Elasticsearch"
  }
  ```

  response

  ```json
  {
    "_index" : "index10",
    "_type" : "_doc",
    "_id" : "1",
    "_version" : 1,
    "result" : "created",
    "_shards" : {
      "total" : 2,
      "successful" : 1,
      "failed" : 0
    },
    "_seq_no" : 0,
    "_primary_term" : 1
  }
  ```

  有兴趣你还可以尝试更多

  ## Operation Type

  索引操作还接受可用于强制创建操作的op_type， 使用create时，如果索引中已存在该id的文档，则索引操作将失败。

  例如：

  request

  ```json
  PUT twitter/_doc/1?op_type=create
  {
      "user" : "kimchy",
      "post_date" : "2009-11-15T14:12:12",
      "message" : "trying out Elasticsearch"
  }
  ```

  response

  ```json
  {
    "error": {
      "root_cause": [
        {
          "type": "version_conflict_engine_exception",
          "reason": "[_doc][1]: version conflict, document already exists (current version [2])",
          "index_uuid": "bvo3_qiWQSq5FOz1KnfTlQ",
          "shard": "3",
          "index": "index10"
        }
      ],
      "type": "version_conflict_engine_exception",
      "reason": "[_doc][1]: version conflict, document already exists (current version [2])",
      "index_uuid": "bvo3_qiWQSq5FOz1KnfTlQ",
      "shard": "3",
      "index": "index10"
    },
    "status": 409
  }
  ```

  `reason`中的异常说明很明显表明，这个文档已存在

  这个表达式的另一种写法是这样的

  ```
  PUT twitter/_doc/1/_create
  {
      "user" : "kimchy",
      "post_date" : "2009-11-15T14:12:12",
      "message" : "trying out Elasticsearch"
  }
  ```

  ## 乐观并发控制

  ## 路由

  默认情况下，通过使用文档的id值的哈希值来控制分片放置 - 或路由。 为了更明确地控制，可以使用路由参数在每个操作的基础上直接指定路由器使用的散列函数的值。 例如：

  ```json
  POST twitter/_doc?routing=kimchy
  {
      "user" : "kimchy",
      "post_date" : "2009-11-15T14:12:12",
      "message" : "trying out Elasticsearch"
  }
  ```

  在上面的示例中，“_ doc”文档根据提供的路由参数`“kimchy”`路由到分片。

  **这个过程是根据下面这个公式决定的：**

  ```
  shard = hash(routing) % number_of_primary_shards
  ```

  routing：就是我们指定的路由参数，通过计算他的hash值，然后对分片总数取余数来定位我们对应的分片位置。

  **这就解释了为什么我们要在创建索引的时候就确定好主分片的数量 并且永远不会改变这个数量：因为如果数量变化了，那么所有之前路由的值都会无效，文档也再也找不到了。**

> ![note](assets/note-1552284648357.png)你可能觉得由于 Elasticsearch 主分片数量是固定的会使索引难以进行扩容。实际上当你需要时有很多技巧可以轻松实现扩容。我们将会在[*扩容设计*](https://www.elastic.co/guide/cn/elasticsearch/guide/current/scale.html)一章中提到更多有关水平扩展的内容。

所有的文档 API（ `get` 、 `index` 、 `delete` 、 `bulk` 、 `update` 以及 `mget` ）都接受一个叫做 `routing` 的路由参数 ，通过这个参数我们可以自定义文档到分片的映射。一个自定义的路由参数可以用来确保所有相关的文档——例如所有属于同一个用户的文档——都被存储到同一个分片中。我们也会在[*扩容设计*](https://www.elastic.co/guide/cn/elasticsearch/guide/current/scale.html)这一章中详细讨论为什么会有这样一种需求。