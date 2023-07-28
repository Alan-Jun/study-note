https://blog.csdn.net/loushuiyifan/article/details/82702522



Hystrix如何实现这些设计目标？

- 使用命令模式将所有对外部服务（或依赖关系）的调用包装在HystrixCommand或HystrixObservableCommand对象中，并将该对象放在单独的线程中执行；
- 每个依赖都维护着一个线程池（或信号量 可以类比java的JUC下面的Semaphore工具类），线程池被耗尽则拒绝请求（而不是让请求排队）。
- 记录请求的成功，失败，超时和线程拒绝。
- 服务错误百分比超过了阈值，熔断器开关自动打开，一段时间内停止对该服务的所有请求。
- 请求失败，被拒绝，超时或熔断时执行降级逻辑。
- 近实时地监控指标和配置的修改。





```

    @HystrixCommand(fallbackMethod = "str_fallbackMethod", groupKey = "strGroupCommand",
            commandKey = "strCommand", threadPoolKey = "strThreadPool",
            commandProperties = {                // 设置隔离策略，THREAD 表示线程池 SEMAPHORE：信号池隔离                
                    @HystrixProperty(name = "execution.isolation.strategy", value = "THREAD"),                // 当隔离策略选择信号池隔离的时候，用来设置信号池的大小（最大并发数）                
                    @HystrixProperty(name = "execution.isolation.semaphore.maxConcurrentRequests", value = "10"),                // 配置命令执行的超时时间

                    @HystrixProperty(name = "execution.isolation.thread.timeoutinMilliseconds", value = "10"),                // 是否启用超时时间

                    @HystrixProperty(name = "execution.timeout.enabled", value = "true"),                // 执行超时的时候是否中断

                    @HystrixProperty(name = "execution.isolation.thread.interruptOnTimeout", value = "true"),                // 执行被取消的时候是否中断               

                    @HystrixProperty(name = "execution.isolation.thread.interruptOnCancel", value = "true"),                // 允许回调方法执行的最大并发数               

                    @HystrixProperty(name = "fallback.isolation.semaphore.maxConcurrentRequests", value = "10"),                // 服务降级是否启用，是否执行回调函数                

                    @HystrixProperty(name = "fallback.enabled", value = "true"),                // 是否启用断路器        
                    @HystrixProperty(name = "circuitBreaker.enabled", value = "true"),                // 该属性用来设置在滚动时间窗中，断路器熔断的最小请求数。例如，默认该值为 20 的时候，                // 如果滚动时间窗（默认10秒）内仅收到了19个请求， 即使这19个请求都失败了，断路器也不会打开。

                    @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold", value = "20"),                // 该属性用来设置在滚动时间窗中，表示在滚动时间窗中，在请求数量超过                // circuitBreaker.requestVolumeThreshold 的情况下，如果错误请求数的百分比超过50,                // 就把断路器设置为 "打开" 状态，否则就设置为 "关闭" 状态。

                    @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage", value = "50"),                // 该属性用来设置当断路器打开之后的休眠时间窗。 休眠时间窗结束之后，                // 会将断路器置为 "半开" 状态，尝试熔断的请求命令，如果依然失败就将断路器继续设置为 "打开" 状态，                // 如果成功就设置为 "关闭" 状态。   

                    @HystrixProperty(name = "circuitBreaker.sleepWindowinMilliseconds", value = "5000"),                // 断路器强制打开     

                    @HystrixProperty(name = "circuitBreaker.forceOpen", value = "false"),                // 断路器强制关闭          

                    @HystrixProperty(name = "circuitBreaker.forceClosed", value = "false"),                // 滚动时间窗设置，该时间用于断路器判断健康度时需要收集信息的持续时间               

                    @HystrixProperty(name = "metrics.rollingStats.timeinMilliseconds", value = "10000"),                // 该属性用来设置滚动时间窗统计指标信息时划分"桶"的数量，断路器在收集指标信息的时候会根据                // 设置的时间窗长度拆分成多个 "桶" 来累计各度量值，每个"桶"记录了一段时间内的采集指标。                // 比如 10 秒内拆分成 10 个"桶"收集这样，所以 timeinMilliseconds 必须能被 numBuckets 整除。否则会抛异常    

                    @HystrixProperty(name = "metrics.rollingStats.numBuckets", value = "10"),                // 该属性用来设置对命令执行的延迟是否使用百分位数来跟踪和计算。如果设置为 false, 那么所有的概要统计都将返回 -1。  

                    @HystrixProperty(name = "metrics.rollingPercentile.enabled", value = "false"),                // 该属性用来设置百分位统计的滚动窗口的持续时间，单位为毫秒。                @HystrixProperty(name = "metrics.rollingPercentile.timeInMilliseconds", value = "60000"),                // 该属性用来设置百分位统计滚动窗口中使用 “ 桶 ”的数量。 

                    @HystrixProperty(name = "metrics.rollingPercentile.numBuckets", value = "60000"),                // 该属性用来设置在执行过程中每个 “桶” 中保留的最大执行次数。如果在滚动时间窗内发生超过该设定值的执行次数，                // 就从最初的位置开始重写。例如，将该值设置为100, 滚动窗口为10秒，若在10秒内一个 “桶 ”中发生了500次执行，                // 那么该 “桶” 中只保留 最后的100次执行的统计。另外，增加该值的大小将会增加内存量的消耗，并增加排序百分位数所需的计算时间。

                    @HystrixProperty(name = "metrics.rollingPercentile.bucketSize", value = "100"),                // 该属性用来设置采集影响断路器状态的健康快照（请求的成功、 错误百分比）的间隔等待时间。                
                    @HystrixProperty(name = "metrics.healthSnapshot.intervalinMilliseconds", value = "500"),                // 是否开启请求缓存   

                    @HystrixProperty(name = "requestCache.enabled", value = "true"),                // HystrixCommand的执行和事件是否打印日志到 HystrixRequestLog 中    

                    @HystrixProperty(name = "requestLog.enabled", value = "true"),},
            threadPoolProperties = {                // 该参数用来设置执行命令线程池的核心线程数，该值也就是命令执行的最大并发量         

                    @HystrixProperty(name = "coreSize", value = "10"),                // 该参数用来设置线程池的最大队列大小。当设置为 -1 时，线程池将使用 SynchronousQueue 实现的队列，                // 否则将使用 LinkedBlockingQueue 实现的队列。 

                    @HystrixProperty(name = "maxQueueSize", value = "-1"),                // 该参数用来为队列设置拒绝阈值。 通过该参数， 即使队列没有达到最大值也能拒绝请求。                // 该参数主要是对 LinkedBlockingQueue 队列的补充,因为 LinkedBlockingQueue                // 队列不能动态修改它的对象大小，而通过该属性就可以调整拒绝请求的队列大小了。               

                    @HystrixProperty(name = "queueSizeRejectionThreshold", value = "5"),})
    public String strConsumer() {
        return "hello 2020";
    }

    public String str_fallbackMethod() {
        return "*****fall back str_fallbackMethod";
    }
```

