# platform-starter-redis使用

## 1. 项目介绍

### 1.1 项目背景

1.    在AWS提供的Redis集群下面，使用Reddision进行红锁加锁有可能锁不住，有两方面的原因：
   * 目前的Redis Cluster的话前面加了一个Proxy，而有可能导致客户端无法直接与redis集群的各个节点进行互相通信导致无法拿取槽信息
   * Redission的红锁必须构建多个RLock来进行并发加锁，且机器的时钟必须同步
     * Reission是默认多RLock一半节点锁住认为锁住，而RLock设置太少的话，有可能只锁在了集群的一个节点上，当一个节点出现问题时，导致锁失败
     * 时钟不同步导致的锁不一致的问题

2. 如果应用中存在Redission和Jedis（Lettuce）多个Redis客户端，对应用来说不好管理，且会导致Redis服务端产生网络风暴的问题

​    在stackoverflow上有一个相关Proxy的讨论 https://stackoverflow.com/questions/31785184/redis-cluster-via-haproxy  

​    Proxy无法获取槽的信息，可以找运维解决此问题

### 1.2 功能列表

-  提供基于RedisTemplate下的红锁功能
-  根据Redis集群的节点数自动构建多个RLock，无须应用自己构建RLock
-  将Redission的策略进行改正，只有在Redis集群多个节点下（可以为Redis集群下所有节点）都加锁成功才返回成功，属于一个悲观策略
-  同一个Key不支持重入



## 2. 依赖引入

- 版本见 [binance-infra-2.0 底层架包介绍](https://confluence.toolsfdg.net/pages/viewpage.action?pageId=38191316)

```
<!--parent 引入这个-->
<parent>
    <groupId>com.binance.infra.avengers</groupId>
    <artifactId>binance-infra-starter-parent</artifactId>
    <version>${infra.version}</version>
    <relativePath />
</parent>
 
 
 
 
<!--jar依赖 -->
<dependency>
     <groupId>com.binance.infra.avengers</groupId>
     <artifactId>platform-starter-redis</artifactId>
</dependency>
```



默认配置

spring.redis.jedis.pool.maxWait=2000 // jedis pool获取资源的最大等待时间，ms

spring.redis.timeout=2000 // jedis connection timeout和read timeout，ms

redis底层pool的默认配置，一般不需要调整

spring.redis.jedis.pool.maxActive=8 //default

spring.redis.jedis.pool.minIdle=0 //default

spring.redis.jedis.pool.maxIdle=8 //default

## 3. 使用示例

### 3.1 Redis分布式锁示例

```
@Autowired
private RedisLock redisLock; //类似于红锁，保证所有节点加锁成功才算成功
 
        //100是等待锁获取时间，3000是锁的expireTime，单位是毫秒
        boolean locked = redisLock.tryLock("your key", 100, 3000);
        try {
            if (locked) {
                // 成功, 处理业务
            } else {
                // 获取锁失败的逻辑
            }
        } finally {
           redisLock.unlock("your key");
        }
```



```
@Autowired
private ConsistentHashJedisLock redisLock;  //类似于一致性hash加锁，性能高
 
        //100是等待锁获取时间，3000是锁的expireTime，单位是毫秒
        boolean locked = redisLock.tryLock("your key", 100, 3000);
        try {
            if (locked) {
                // 成功, 处理业务
            } else {
                // 获取锁失败的逻辑
            }
        } finally {
            if(locked) {
                redisLock.unlock("your key");
            }
        }
```

### 3.2 Redis cluster pipeline示例

pipeline操作旨在减少网络开销，在特定场景下能大幅提高redis的使用效率。因此基于Jedis实现了支持cluster的pipeline。

```
 private ThreadPoolExecutor parallelPool; //线程池核心线程数量大于等于redis分片数，可以提高最高性能。
    @Autowired
    private JedisClusterPipeline jedisClusterPipeline;
     
    @PostConstruct
    private void initPool() {
        parallelPool = new ThreadPoolExecutor(15, 15, 1, TimeUnit.MINUTES,
                new ArrayBlockingQueue<>(100),
                new ThreadFactoryBuilder().setNameFormat("redis-concurrent-pool-%d").build(),
                new ThreadPoolExecutor.CallerRunsPolicy());
    }    
 
//key的crc16在相同slot示例
        TestBo vo = new TestBo();
        vo.setUserId(12333L);
        vo.setAsset("spot");
 
        PipelineCmdBuilder builder = new PipelineCmdBuilder(redisTemplate);
        builder.set("testKey1", vo);
        builder.expire("testKey1", 10);
        builder.set("prefix:{testKey1}:buy", 10);
        builder.set("prefix:{testKey1}:sell", 10);
        builder.expire("prefix:{testKey1}:sell", 10);
        List<PipelineCmd.PipelineHolder> opsList = builder.build();
        jedisClusterPipeline.sameSlotPipeline(opsList);
         
        //多线程并发示例
        builder = new PipelineCmdBuilder(redisTemplate);
        builder.set("myTest6", JsonUtils.toJsonNotNullKey(vo));
        builder.set("myTest7", userAsset);
        builder.get("myTest1");
        builder.get("myTest2");
        builder.zrangeWithScores("myTest5", 0, -1);
 
        for (Long uid : userIdList) {
            builder.get(uidPrefix + uid);
        }
        opsList =builder.build();
        jedisClusterPipeline.parallelPipelineCall(opsList, parallelPool, 500);
        // 根据需要获取结果
        List<Object> retList = new ArrayList<>();
        for (PipelineCmd.PipelineHolder holder : opsList) {
            retList.add(holder.getResult());
        }
```

使用须知：

1. JedisClusterPipeline目前只支持redis cluster模式，并配置spring.redis.cluster.nodes时，才有效。
2. JedisClusterPipeline和RedisTemplate是兼容的。RedisTemplate写入的数据，JedisClusterPipeline可以获取，反之成立。
   如果RedisTemplate是服务手动new的，并且未使用基础包的序列化方式，则不保证兼容性。
3. 建议spring.redis.timeout设置一个合理的数值，比如200，这样在失败时可以自动快速重试。如果数值过小导致本地启动报错，请在dev和qa环境指定较大值。



### 3.3 Redis cluster读写分离

我们redis cluster的部署方式一般每个分片是一个master和一个slave，默认行为是读写都会发生在master上，slave只和master同步数据不对外提供服务。如果开启读写分离，并且每个分片配置多个slave节点，可以扩展整个集群的读性能。

**使用方法：**

1. 开启读写分离配置spring.redis.cluster.readWriteSplit=true，这样所有的读请求默认都会从slave读，需要注意这样可能会引入一定的数据不一致。
2. 提供spring.redis.cluster.readOnMaster配置，默认为false，打开后可以让主节点也可以参与读操作。
3. 提供了类似mybatis的HintManager，可以通过使用HintManager来指定从master节点读取数据。

```
HintManager.getInstance().preferReadOnMaster();``//... your redis code here, either use RedisTemplate or directly use JedisCluster``HintManager.clear();
```