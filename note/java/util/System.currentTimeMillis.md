# System.currentTimeMillis(); 高频调用存在严重的性能问题：因为它的底层调用是：

- 调用gettimeofday()需要从用户态切换到内核态；
- gettimeofday()的表现受Linux系统的计时器（时钟源）影响，在HPET计时器下性能尤其差；
- 系统只有一个全局时钟源，高并发或频繁访问会造成严重的争用。

所以我们在使用的时候需要做特殊处理，比如：这段代码是从 sentinel的源码中直接copy来的

```java
public final class TimeUtil {
    private static volatile long currentTimeMillis = System.currentTimeMillis();

    public TimeUtil() {
    }

    public static long currentTimeMillis() {
        return currentTimeMillis;
    }

    static {
      // 使用一个守护线程去没毫秒去更新它的数值，并且通过该工具解决底层被并发调用的问题
        Thread daemon = new Thread(new Runnable() {
            public void run() {
                while(true) {
                    TimeUtil.currentTimeMillis = System.currentTimeMillis();

                    try {
                        TimeUnit.MILLISECONDS.sleep(1L);
                    } catch (Throwable var2) {
                    }
                }
            }
        });
        daemon.setDaemon(true);
        daemon.setName("time-tick-thread");
        daemon.start();
    }
}
```

# 