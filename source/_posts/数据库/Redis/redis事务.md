---
title: Redis 事务
tags:
  - Redis
categories:
  - 数据库
---
# Redis 事务
 Redis事务的分析及改进：https://blog.csdn.net/kingmax54212008/article/details/82731199
 
 ## SpringBoot Redis 事务
 **配置类**
 ```java
@Configuration
@EnableTransactionManagement//开启事务
public class BeanConfig {

    @Bean
    public RedisTemplate<String, Object> redisTemplate() {//创建RedisTemplate
        // 配置redisTemplate
        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(lettuceConnectionFactory);
        setRedisTemplate(redisTemplate);
        //开启事务
        redisTemplate.setEnableTransactionSupport(true);
        return redisTemplate;
    }
}
```

**业务代码**
```java
/**
 * @author zc
 * @date 2020/9/15 20:12
 */
@Slf4j
@Service
public class TransactionService {

    @Autowired
    private RedisTemplate<String, Object> redisTemplate;

    @Transactional(rollbackFor = Exception.class)
    public void test1() throws Exception {
        System.out.println(redisTemplate.opsForValue().get("test1"));
        redisTemplate.opsForValue().set("test1", "test1");
        throw new Exception("异常信息");
    }

    /**
     * 无需配置，手动开启事务
     *
     * @param flag 是否报错
     * @param key  key
     * @throws Exception 异常
     */
    public void test2(boolean flag, String key) throws Exception {
        redisTemplate.multi();

        redisTemplate.opsForValue().set(key, key);
        if (flag) {
            throw new Exception("异常信息");
        }
        redisTemplate.opsForValue().set(key + 2, key + 2);

        redisTemplate.exec();
    }
}

```

**测试代码**
```java
@SpringBootTest
class RedisTransactionApplicationTests {

    @Autowired
    TransactionService transactionService;

    @Autowired
    private RedisTemplate<String, Object> redisTemplate;

    @Test
    void redisKey() throws Exception {
//        transactionService.test1();
//        transactionService.test2(false, "key1");
        transactionService.test2(true, "key2");

//        System.out.println("test1: " + redisTemplate.opsForValue().get("test1"));
//        System.out.println("key11: " + redisTemplate.opsForValue().get("key11"));
//        System.out.println("key12: " + redisTemplate.opsForValue().get("key12"));
//
//        System.out.println("key21: " + redisTemplate.opsForValue().get("key21"));
//        System.out.println("key22: " + redisTemplate.opsForValue().get("key22"));
    }
}

```
