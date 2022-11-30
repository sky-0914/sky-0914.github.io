---
title: redis监听缓存失效通知事件
tags:
  - Redis
categories:
  - 数据库
---
# redis监听缓存失效通知事件

## Redis配置

**先修改Redis配置**
>修改redis相关事件配置。找到redis配置文件redis.conf，查看“notify-keyspace-events”的配置项，如果没有，添加“notify-keyspace-events Ex”，如果有值，添加Ex，然后重启Redis。相关参数说明如下：
```
K：keyspace事件，事件以__keyspace@<db>__为前缀进行发布；         
E：keyevent事件，事件以__keyevent@<db>__为前缀进行发布；         
g：一般性的，非特定类型的命令，比如del，expire，rename等；        
$：字符串特定命令；         
l：列表特定命令；         
s：集合特定命令；         
h：哈希特定命令；         
z：有序集合特定命令；         
x：过期事件，当某个键过期并删除时会产生该事件；         
e：驱逐事件，当某个键因maxmemore策略而被删除时，产生该事件；         
A：g$lshzxe的别名，因此”AKE”意味着所有事件。
```
**redis客户端测试：**

客户端1：
```shell script
127.0.0.1:6379> psubscribe __keyevent@0__:expired
Reading messages... (press Ctrl-C to quit)
1) "psubscribe"
2) "__keyevent@0__:expired"
3) (integer) 1
```
`keyevent@0:expired`解析：
+ key 键
+ event 事件
+ @0 第0个数据库
+ expired 过期

客户端2：
```shell script
127.0.0.1:6379> set a 111 ex 5
```
客户端1：
```shell script
1) "pmessage"
2) "__keyevent@0__:expired"
3) "__keyevent@0__:expired"
4) "a"
```

## java代码

**配置Redis监听类**
```java
@Slf4j
@Configuration
public class BeanConfig {

    /**
     * 配置Redis监听类
     * 与Spring Redis Session 冲突
     * springSessionRedisMessageListenerContainer: defined by method 'springSessionRedisMessageListenerContainer' in class path resource [org/springframework/boot/autoconfigure/session/RedisSessionConfiguration$SpringBootRedisHttpSessionConfiguration.class]
     *
     * @param connectionFactory
     * @return
     */
    @Bean
    public RedisMessageListenerContainer container(RedisConnectionFactory connectionFactory) {
        RedisMessageListenerContainer container = new RedisMessageListenerContainer();
        container.setConnectionFactory(connectionFactory);
//        container.addMessageListener(new RedisExpiredListener(), new PatternTopic("__keyevent@0__:expired"));
        return container;
    }

```
**Redis监听类**
```java
/**
 * Redis监听类
 *
 * @author zc
 * @date 2020/9/15 15:12
 */
@Slf4j
@Component
public class ListenRedisKeyFailure extends KeyExpirationEventMessageListener {
    public ListenRedisKeyFailure(RedisMessageListenerContainer listenerContainer) {
        super(listenerContainer);
    }

    /**
     * 配置监听哪个频道
     * 我们在配置文件中配置的是8数据库，所以监听8数据库
     */
    private static final Topic KEYEVENT_EXPIRED_TOPIC = new PatternTopic("__keyevent@8__:expired");

    @Override
    protected void doRegister(RedisMessageListenerContainer listenerContainer) {
        // 频道可以是多，多个传list
        listenerContainer.addMessageListener(this, KEYEVENT_EXPIRED_TOPIC);
    }

    /**
     * 针对redis数据失效事件，进行数据处理
     * message：Redis失效的Key，无法获取value值
     *
     * @param message 失效key
     * @param pattern
     */
    @Override
    public void onMessage(Message message, byte[] pattern) {
        // 用户做自己的业务处理即可,注意message.toString()可以获取失效的key
        String expiredKey = message.toString();
        if (expiredKey.startsWith("test:")) {
            //如果是Order:开头的key，进行处理
            log.info("Redis 过期的 Key：{}", expiredKey);
        }
    }
}
```

**测试代码**
```java
@SpringBootTest
class RedisListenKeyFailureApplicationTests {

    @Autowired
    RedisTemplate<String, Object> redisTemplate;

    @Test
    void redisKey() throws InterruptedException {
        //设置缓存，2秒过期
        redisTemplate.opsForValue().set("test:1", "", 2, TimeUnit.SECONDS);
        //程序等待5秒
        Thread.sleep(50000);
    }
}
```
