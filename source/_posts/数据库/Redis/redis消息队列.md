---
title: Redis 消息队列
tags:
  - Redis
categories:
  - 数据库
---
# Redis 消息队列
```java
/**
 * 消息队列
 */
@SpringBootTest
class RedisQueueApplicationTests {

    @Autowired
    RedisTemplate<String, Object> redisTemplate;

    /**
     * Redis消息队列利用List来实现
     * 通过向列表左或者右push数据,之后在通过左或者右pop弹出数据来实现
     */
    @Test
    void redisQueue() {
        for (int i = 0; i < 10; i++) {
            //向key为test左边push数据
            redisTemplate.opsForList().leftPush("test", i);
//            redisTemplate.opsForList().rightPush("test", i);
        }
        for (int i = 0; i < 10; i++) {
            //从key为test左边pop弹出数据
            System.out.println(redisTemplate.opsForList().leftPop("test"));
//            System.out.println(redisTemplate.opsForList().rightPop("test"));
        }
    }
}

```
