---
title: Redis 分布式锁
tags:
  - Redis
categories:
  - 数据库
---
# Redis 分布式锁
Redis的setnx命令是当key不存在时设置key，但setnx不能同时完成expire设置失效时长，不能保证setnx和expire的原子性。我们可以使用set命令完成setnx和expire的操作，并且这种操作是原子操作。
下面是set命令的可选项：
```shell
set key value [EX seconds] [PX milliseconds] [NX|XX]
EX seconds：设置失效时长，单位秒
PX milliseconds：设置失效时长，单位毫秒
NX：key不存在时设置value，成功返回OK，失败返回(nil)
XX：key存在时设置value，成功返回OK，失败返回(nil)

案例：设置name=p7+，失效时长100s，不存在时设置
1.1.1.1:6379> set name p7+ ex 100 nx
OK
1.1.1.1:6379> get name
"p7+"
1.1.1.1:6379> ttl name
(integer) 94
```

**实现代码**
```java
/**
 * 分布式锁
 *
 * @author zc
 * @date 2020/9/15 11:17
 */
@Slf4j
@Service
public class DistributedLockService {

    @Autowired
    RedisTemplate<String, Object> redisTemplate;

    /**
     * 加锁
     *
     * @param key   键
     * @param value 值
     * @return 返回值
     */
    private Boolean lock(String key, Object value) {
        return redisTemplate.opsForValue().setIfAbsent(key, value, 10, TimeUnit.SECONDS);
    }

    /**
     * 解锁
     *
     * @param key 键
     * @return 返回值
     */
    private Boolean unLock(String key) {
        return redisTemplate.delete(key);
    }

    public Long incr(String key) {
        return redisTemplate.opsForValue().increment(key, 1);
    }

    public Long distributedLock(String goodsId) {
        int i = 1;
        while (true) {
            log.info(goodsId + "," + i++);
            if (this.lock(goodsId, goodsId)) {
                Long l = this.incr("incr_" + goodsId);
                log.info("递增数量：{}", l);
                this.unLock(goodsId);
                return l;
            }
        }
    }
}
```

**测试代码**
```java
@Slf4j
@SpringBootTest
class RedisDistributedLockServiceApplicationTests {

    @Autowired
    DistributedLockService distributedLockService;

    @Test
    void redisDistributedLock() throws InterruptedException {
        for (int i = 0; i < 10; i++) {
            int a = i;
            Thread thread = new Thread("lock") {
                @Override
                public void run() {
                    log.info("DistributedLock: {}", distributedLockService.distributedLock("goodsId"));
                }
            };
            thread.start();
        }
        Thread.sleep(20000);
    }
}

```
