---
title: 【SpringBoot2】整合Redis缓存
tags:
  - SpringBoot
  - Redis
  - 缓存
categories:
  - SpringBoot
---
遵循SpringBoot三板斧
**第一步加依赖**
```pom
<!-- Redis -->
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<!-- redis依赖commons-pool 这个依赖一定要添加 -->
<dependency>
	<groupId>org.apache.commons</groupId>
	<artifactId>commons-pool2</artifactId>
	<version>2.6.0</version>
</dependency>
```
**第二步写注解**
```java
@EnableCaching//开启缓存支持
```
**第三步写配置**
```yml
spring:
  redis:
    database: 0
    host: 192.168.1.11
    port: 6379
    password:
    timeout: 600
    lettuce:
      pool:
        max-active: 50
        max-wait: -1
        max-idle: 8
        min-idle: 0
```
**编写Redis配置类**
```java
/**
 * @Author: zc
 * @Date: 2019/11/3 14:12
 * @Description: SpringBoot2.0 Redis缓存配置
 * @EnableCaching:开启缓存支持
 */
@Slf4j
@Configuration
@EnableCaching
public class RedisConfig extends CachingConfigurerSupport {

    @Value("${sys.dataCaching.expireTime:0}")
    private int expireTime;

    @Resource
    private LettuceConnectionFactory lettuceConnectionFactory;

    @Override
    @Bean
    public KeyGenerator keyGenerator() {//设置自定义key{ClassName + methodName + params}
        return (target, method, params) -> {
            StringBuilder sb = new StringBuilder();
            sb.append(target.getClass().getName());
            sb.append(",Method:");
            sb.append(method.getName());
            sb.append(",Params[");
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
	//自定义keyGenerator，Key生成器
	@Bean
    public KeyGenerator updateByIdkeyGenerator() {
        return (target, method, params) -> {
            StringBuilder sb = new StringBuilder();
            sb.append(target.getClass().getName());
            sb.append(",Method:");
            sb.append("getById");
            sb.append(",Params[");
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
	//自定义keyGenerator，Key生成器
    @Bean
    public KeyGenerator deleteByIdkeyGenerator() {
        return (target, method, params) -> {
            StringBuilder sb = new StringBuilder();
            sb.append(target.getClass().getName());
            sb.append(",Method:");
            sb.append("getById");
            sb.append(",Params[");
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
        //设置缓存过期时间
        if (expireTime > 0) {
            log.info("Redis 缓存过期时间 : {}", expireTime);
            //设置缓存有效期 秒
            redisCacheConfiguration.entryTtl(Duration.ofSeconds(expireTime));
        } else {
            log.info("Redis 未设置缓存过期时间");
        }
        return RedisCacheManager
                .builder(RedisCacheWriter.nonLockingRedisCacheWriter(redisConnectionFactory))
                .cacheDefaults(redisCacheConfiguration).build();
    }

    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory) {//创建RedisTemplate
        // 设置序列化
        Jackson2JsonRedisSerializer<Object> jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer<Object>(
                Object.class);
        ObjectMapper om = new ObjectMapper();
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(om);
        // 配置redisTemplate
        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<String, Object>();
        redisTemplate.setConnectionFactory(lettuceConnectionFactory);
        RedisSerializer<?> stringSerializer = new StringRedisSerializer();
        // key序列化
        redisTemplate.setKeySerializer(stringSerializer);
        // value序列化
        redisTemplate.setValueSerializer(jackson2JsonRedisSerializer);
        // Hash key序列化
        redisTemplate.setHashKeySerializer(stringSerializer);
        // Hash value序列化
        redisTemplate.setHashValueSerializer(jackson2JsonRedisSerializer);
        redisTemplate.afterPropertiesSet();
        return redisTemplate;
    }
}
```
**如何使用查询缓存**
```java
@CacheConfig(cacheNames = "demoDao")
@Component
public class DemoDao implements IDemoDAO<> {
    @Autowired
    DemoMapper mapper;

	//用默认配置的keyGenerator
    @Cacheable
    @Override
    public Demo getById(Integer id) {
        return mapper.getById(id);
    }
	//使用配置的keyGenerator，清空缓存
	@CacheEvict(keyGenerator = "updateByIdkeyGenerator")
    @Override
    public int update(T entity) {
        return mapper.update(entity);
    }
	//使用配置的keyGenerator，清空缓存
	@CacheEvict(keyGenerator = "deleteByIdkeyGenerator")
    @Override
    public int deleteById(Integer id) {
        return mapper.deleteById(id);
    }
}
```
