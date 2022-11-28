---
title: 【SpringBoot2】整合Redis多数据源
tags:
  - SpringBoot
  - Redis
categories:
  - SpringBoot
---
**配置文件属性**
```yaml
spring:
  redis:
    database: 1
    host: 192.168.50.144
    port: 6379
    password:
    timeout: 600 #Springboot2.0 不能设置为0
    lettuce:
      pool:
        max-active: 50
        max-wait: -1
        max-idle: 8
        min-idle: 0
  redis2:
    database: 2
    host: 192.168.50.144
    port: 6379
    password:
    timeout: 600
```
**配置类**
```java
@EnableCaching
@Configuration
public class RedisDevConfiguration {
    @Bean(name = "redisDevTemplate")
    public StringRedisTemplate redisTemplate(@Value("${spring.redis.host}") String hostName,
                                             @Value("${spring.redis.port}") int port, @Value("${spring.redis.password}") String password,
                                             @Value("${spring.redis.lettuce.pool.max-idle}") int maxIdle, @Value("${spring.redis.lettuce.pool.max-active}") int maxTotal,
                                             @Value("${spring.redis.database}") int index, @Value("${spring.redis.lettuce.pool.max-wait}") long maxWaitMillis,@Value("${spring.redis.lettuce.pool.min-idle}") int minIdle) {
        StringRedisTemplate temple = new StringRedisTemplate();
        temple.setConnectionFactory(
                connectionFactory(hostName, port, password, maxIdle, maxTotal, index, maxWaitMillis,minIdle));

        return temple;
    }


    @Bean(name = "redisUatTemplate")
    public StringRedisTemplate redisUatTemplate(@Value("${spring.redis2.host}") String hostName,
                                             @Value("${spring.redis2.port}") int port, @Value("${spring.redis2.password}") String password,
                                             @Value("${spring.redis.lettuce.pool.max-idle}") int maxIdle, @Value("${spring.redis.lettuce.pool.max-active}") int maxTotal,
                                             @Value("${spring.redis2.database}") int index, @Value("${spring.redis.lettuce.pool.max-wait}") long maxWaitMillis,@Value("${spring.redis.lettuce.pool.min-idle}") int minIdle) {
        StringRedisTemplate temple = new StringRedisTemplate();
        temple.setConnectionFactory(
                connectionFactory(hostName, port, password, maxIdle, maxTotal, index, maxWaitMillis,minIdle));

        return temple;
    }

    public RedisConnectionFactory connectionFactory(String hostName, int port, String password, int maxIdle,
                                                    int maxTotal, int index, long maxWaitMillis,int minIdle) {
        JedisConnectionFactory jedis = new JedisConnectionFactory();
        jedis.setHostName(hostName);
        jedis.setPort(port);
        if (StringUtils.isNotEmpty(password)) {
            jedis.setPassword(password);
        }
        if (index != 0) {
            jedis.setDatabase(index);
        }
        jedis.setPoolConfig(poolCofig(maxIdle, maxTotal, maxWaitMillis,minIdle));
        // 初始化连接pool
        jedis.afterPropertiesSet();
        RedisConnectionFactory factory = jedis;

        return factory;
    }

    public JedisPoolConfig poolCofig(int maxIdle, int maxTotal, long maxWaitMillis,int minIdle) {
        JedisPoolConfig poolCofig = new JedisPoolConfig();
        poolCofig.setMaxIdle(maxIdle);
        poolCofig.setMaxTotal(maxTotal);
        poolCofig.setMaxWaitMillis(maxWaitMillis);
        poolCofig.setMinIdle(minIdle);
        return poolCofig;
    }
}
```
**如何使用**
```java
	@Resource(name = "redisDevTemplate")
	private StringRedisTemplate template;

	@Resource(name = "redisUatTemplate")
	private StringRedisTemplate lockTemplate;
```
