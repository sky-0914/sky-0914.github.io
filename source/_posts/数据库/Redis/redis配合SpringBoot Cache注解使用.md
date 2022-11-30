---
title: redis配合SpringBoot Cache注解使用
tags:
  - Redis
categories:
  - 数据库
---
## Cache配置类

```java
/**
 * Cache配置类
 *
 * @author zc
 * @date 2020/9/14 20:10
 */
@Slf4j
@EnableCaching//开启注解缓存
@Configuration
public class CustomKeyGenerator extends CachingConfigurerSupport {

    /**
     * 设置自定义key{ClassName + methodName + params}
     *
     * @return
     */
    @Bean
    @Override
    public KeyGenerator keyGenerator() {
        return (target, method, params) -> {
            StringBuilder sb = new StringBuilder();
            sb.append(target.getClass().getName());
            sb.append(":Method:");
//            sb.append("Method:");
            sb.append(method.getName());
            StringJoiner sj = new StringJoiner(",", ":Params[", "]");
            for (Object param : params) {
                if (param instanceof Array) {
                    sj.add(param.toString());
                }
                sj.add(param.toString());
            }
            sb.append(sj.toString());
            log.debug("Data Caching Redis Key : {}", sb.toString());
            return sb.toString();
        };
    }

    @Bean
    public KeyGenerator saveGenerator() {
        return (target, method, params) -> {
            StringBuilder sb = new StringBuilder();
            sb.append(target.getClass().getName());
            sb.append(":Method:");
            sb.append("getOne");
            sb.append(":Params[");
            try {
                Field id = params[0].getClass().getDeclaredField("id");
                id.setAccessible(true);
                sb.append(id.get(params[0]).toString());
            } catch (IllegalAccessException e) {
                e.printStackTrace();
            } catch (NoSuchFieldException e) {
                e.printStackTrace();
            }
            sb.append("]");
            log.debug("Data Caching Redis Key : {}", sb.toString());
            return sb.toString();
        };
    }

    @Bean
    public KeyGenerator deleteByIdGenerator() {
        return (target, method, params) -> {
            StringBuilder sb = new StringBuilder();
            sb.append(target.getClass().getName());
            sb.append(":Method:");
            sb.append("getById");
            sb.append(":Params[");
            for (int i = 0; i < params.length; i++) {
                sb.append(params[i].toString());
                if (i != (params.length - 1)) {
                    sb.append(",");
                }
            }
            sb.append("]");
            log.debug("Data Caching Redis Key : {}", sb.toString());
            return sb.toString();
        };
    }

    @Bean
    public CacheManager cacheManager(RedisConnectionFactory redisConnectionFactory) {
        RedisCacheConfiguration redisCacheConfiguration = RedisCacheConfiguration.defaultCacheConfig();
        int expireTime = 0;
        //设置缓存过期时间
        if (expireTime > 0) {
            log.info("Redis 缓存过期时间 : {}", expireTime);
            //设置缓存有效期 秒
            redisCacheConfiguration = redisCacheConfiguration.entryTtl(Duration.ofSeconds(expireTime));
        } else {
            log.info("Redis 未设置缓存过期时间");
        }
        redisCacheConfiguration = redisCacheConfiguration.serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(new StringRedisSerializer()))
                .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(new GenericJackson2JsonRedisSerializer()))
                .disableCachingNullValues();
        return RedisCacheManager.builder(redisConnectionFactory).cacheDefaults(redisCacheConfiguration).build();
    }
}
```

## 编写业务代码
```java
/**
 * @author zc
 * @date 2020/9/14 19:49
 */
@CacheConfig(cacheNames = {"cache"})
@Slf4j
@Service
public class CacheService {

    private List<StudentDTO> list;

    public CacheService() {
        log.info("实例化数组");
        this.list = new ArrayList<>();
        list.add(StudentDTO.builder()
                .id(1L)
                .age(18)
                .name("张三")
                .build());
        list.add(StudentDTO.builder()
                .id(2L)
                .age(20)
                .name("李四")
                .build());
    }

    @Cacheable
    public List<StudentDTO> getAll() {
        log.info("查询数据");
        return list;
    }

    @Cacheable
    public List<StudentDTO> getOne(Long id) {
        log.info("查询单条数据");
        return list.stream().filter(s -> s.getId().equals(id)).collect(Collectors.toList());
    }

    @CacheEvict(keyGenerator = "saveGenerator")
    public List<StudentDTO> save(StudentDTO dto) {
        log.info("保存数据");
        List<StudentDTO> newList = new ArrayList<>();
        list.forEach(s -> {
            if (s.getId().equals(dto.getId())) {
                newList.add(dto);
            } else {
                newList.add(s);
            }
        });
        this.list = newList;
        return list;
    }
}

```

## 编写测试代码
```java
@SpringBootTest
class RedisCacheApplicationTests {

    @Autowired
    CacheService cacheService;

    /**
     * 查询数据
     * 2020-09-14 20:50:10.650  INFO 57271 --- [           main] cn.happyloves.redis.cache.CacheService   : 查询数据
     * [StudentDTO(id=1, name=张三, age=18), StudentDTO(id=2, name=李四, age=20)]
     * 2020-09-14 20:50:10.819  INFO 57271 --- [           main] cn.happyloves.redis.cache.CacheService   : 查询单条数据
     * [StudentDTO(id=1, name=张三, age=18)]
     * 2020-09-14 20:50:10.886  INFO 57271 --- [           main] cn.happyloves.redis.cache.CacheService   : 查询单条数据
     * [StudentDTO(id=2, name=李四, age=20)]
     * [StudentDTO(id=1, name=张三, age=18), StudentDTO(id=2, name=李四, age=20)]
     * [StudentDTO(id=1, name=张三, age=18)]
     * [StudentDTO(id=2, name=李四, age=20)]
     * 2020-09-14 20:50:11.118  INFO 57271 --- [           main] cn.happyloves.redis.cache.CacheService   : 保存数据
     * [StudentDTO(id=1, name=张三, age=18), StudentDTO(id=2, name=李四, age=22)]
     * 2020-09-14 20:50:11.205  INFO 57271 --- [           main] cn.happyloves.redis.cache.CacheService   : 查询单条数据
     * [StudentDTO(id=2, name=李四, age=22)]
     */
    @Test
    void redisCache() {
        System.out.println(cacheService.getAll());
        System.out.println(cacheService.getOne(1L));
        System.out.println(cacheService.getOne(2L));

        System.out.println(cacheService.getAll());
        System.out.println(cacheService.getOne(1L));
        System.out.println(cacheService.getOne(2L));


        System.out.println(cacheService.save(StudentDTO.builder()
                .id(2L)
                .age(22)
                .name("李四")
                .build()));
        System.out.println(cacheService.getOne(2L));
    }
}
```
