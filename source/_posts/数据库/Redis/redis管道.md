---
title: Redis 管道 Pipeline
tags:
  - Redis
categories:
  - 数据库
---
# Redis 管道 Pipeline

**JAVA代码**
```java
/**
 * 管道，批量操作
 *
 * @author zc
 * @date 2020/9/15 18:46
 */
@Service
public class PipelineService {
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;

    public void pipeline() {
        redisTemplate.executePipelined((RedisCallback<String>) connection -> {
            for (int i = 0; i < 100; i++) {
                connection.set(("pipeline:" + i).getBytes(), "123".getBytes());
            }
            return null;
        });
    }

    public void common() {
        for (int i = 0; i < 100; i++) {
            redisTemplate.opsForValue().set("common:" + i, "123");
        }
    }
}
```
**测试代码**
```java
@SpringBootTest
class RedisPipelineApplicationTests {

    @Autowired
    PipelineService pipelineService;

    /**
     * 测试管道和普通的速度
     * pipeline: 272ms
     * common: 6859ms
     */
    @Test
    void redisPipeline() {
        long t1 = System.currentTimeMillis();
        pipelineService.pipeline();
        long t2 = System.currentTimeMillis();
        pipelineService.common();
        long t3 = System.currentTimeMillis();
        System.out.println("pipeline: " + (t2 - t1) + "ms");
        System.out.println("common: " + (t3 - t2) + "ms");
    }
}
```
