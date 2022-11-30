---
title: Redis 发布订阅模式MQ
tags:
  - Redis
categories:
  - 数据库
---
# Redis 发布订阅模式MQ

**常量主题名称**
```java
/**
 * @author zc
 * @date 2020/9/16 00:02
 */
public class Constant {

    public static final String MQ_TOPIC_TEST1 = "topic-test1";
    public static final String MQ_TOPIC_TEST2 = "topic-test2";
    public static final String MQ_TOPIC_TEST3 = "topic-test3";
    public static final String MQ_TOPIC_TEST4 = "topic-test4";
    public static final String MQ_TOPIC_TEST5 = "topic-test5";
}
```

**处理器**
```java
/**
 * @author zc
 * @date 2020/9/16 00:02
 */
@Slf4j
@Component
public class ReceiverRedisMessage {

    public void test1Listener(String msg) {
        log.info("[1====开始消费REDIS消息队列TOPIC_TEST1数据...],消息数据[{}]", msg);
    }

    public void test2Listener(String msg) {
        log.info("[2====开始消费REDIS消息队列TOPIC_TEST2数据...],消息数据[{}]", msg);
    }

    public void test3Listener(Object msg) {
        log.info("[3====开始消费REDIS消息队列TOPIC_TEST3数据...],消息数据[{}]", msg.toString());
    }

    public void test4Listener(Object msg) {
        log.info("[4====开始消费REDIS消息队列TOPIC_TEST4数据...],消息数据[{}]", msg.toString());
    }

    public void test5Listener(Object msg) {
        log.info("[5====开始消费REDIS消息队列TOPIC_TEST5数据...],消息数据[{}]", msg.toString());
    }
}

```


**配置类**
```java
/**
 * @author zc
 * @date 2020/9/16 00:02
 */
@Configuration
public class MqBeanConfig {


    /**
     * redis消息监听器容器
     * 可以添加多个监听不同话题的redis监听器，只需要把消息监听器和相应的消息订阅处理器绑定，该消息监听器
     * 通过反射技术调用消息订阅处理器的相关方法进行一些业务处理
     *
     * @param connectionFactory
     * @param test1ListenerAdapter
     * @param test2ListenerAdapter
     * @param test3ListenerAdapter
     * @param test4ListenerAdapter
     * @return
     */
    @Bean("container")
    RedisMessageListenerContainer container(RedisConnectionFactory connectionFactory,
                                            MessageListenerAdapter test1ListenerAdapter,
                                            MessageListenerAdapter test2ListenerAdapter,
                                            MessageListenerAdapter test3ListenerAdapter,
                                            MessageListenerAdapter test4ListenerAdapter,
                                            MessageListenerAdapter test5ListenerAdapter) {
        RedisMessageListenerContainer container = new RedisMessageListenerContainer();
        container.setConnectionFactory(connectionFactory);
        //监听TEST1情况主题并绑定消息订阅处理器
        container.addMessageListener(test1ListenerAdapter, new PatternTopic(Constant.MQ_TOPIC_TEST1));
        //监听TEST2主题并绑定消息订阅处理器
        container.addMessageListener(test2ListenerAdapter, new PatternTopic(Constant.MQ_TOPIC_TEST2));
        //监听TEST3主题并绑定消息订阅处理器
        container.addMessageListener(test3ListenerAdapter, new PatternTopic(Constant.MQ_TOPIC_TEST3));
        //监听TEST4主题并绑定消息订阅处理器
        container.addMessageListener(test4ListenerAdapter, new PatternTopic(Constant.MQ_TOPIC_TEST4));
        //监听TEST5主题并绑定消息订阅处理器,监听器同时订阅多个主题
        container.addMessageListener(test5ListenerAdapter, new PatternTopic(Constant.MQ_TOPIC_TEST1));
        container.addMessageListener(test5ListenerAdapter, new PatternTopic(Constant.MQ_TOPIC_TEST2));
        container.addMessageListener(test5ListenerAdapter, new PatternTopic(Constant.MQ_TOPIC_TEST3));
        container.addMessageListener(test5ListenerAdapter, new PatternTopic(Constant.MQ_TOPIC_TEST4));
        return container;
    }

    @Bean
    MessageListenerAdapter test1ListenerAdapter(ReceiverRedisMessage receiver) {
        return new MessageListenerAdapter(receiver, "test1Listener");
    }

    @Bean
    MessageListenerAdapter test2ListenerAdapter(ReceiverRedisMessage receiver) {
        return new MessageListenerAdapter(receiver, "test2Listener");
    }

    @Bean
    MessageListenerAdapter test3ListenerAdapter(ReceiverRedisMessage receiver) {
        return new MessageListenerAdapter(receiver, "test3Listener");
    }

    @Bean
    MessageListenerAdapter test4ListenerAdapter(ReceiverRedisMessage receiver) {
        return new MessageListenerAdapter(receiver, "test4Listener");
    }

    @Bean
    MessageListenerAdapter test5ListenerAdapter(ReceiverRedisMessage receiver) {
        return new MessageListenerAdapter(receiver, "test5Listener");
    }
}
```

**测试代码**
```java
@SpringBootTest
class RedisMqApplicationTests {

    @Autowired
    RedisTemplate<String, Object> redisTemplate;

    @Test
    void redisMq() {
        redisTemplate.convertAndSend(Constant.MQ_TOPIC_TEST1, "111111");
        redisTemplate.convertAndSend(Constant.MQ_TOPIC_TEST2, "222222");
        TestVO tvo1 = new TestVO();
        tvo1.setName("张三");
        redisTemplate.convertAndSend(Constant.MQ_TOPIC_TEST3, tvo1);
        TestVO tvo2 = new TestVO();
        tvo2.setName("李四");
        redisTemplate.convertAndSend(Constant.MQ_TOPIC_TEST4, tvo2);
    }
}

```

**运行结果**
```
2020-09-16 01:04:10.317  INFO 9741 --- [    container-2] c.h.r.p.subscribe.ReceiverRedisMessage   : [1====开始消费REDIS消息队列TOPIC_TEST1数据...],消息数据["111111"]
2020-09-16 01:04:10.317  INFO 9741 --- [    container-3] c.h.r.p.subscribe.ReceiverRedisMessage   : [5====开始消费REDIS消息队列TOPIC_TEST5数据...],消息数据["111111"]
2020-09-16 01:04:10.343  INFO 9741 --- [    container-4] c.h.r.p.subscribe.ReceiverRedisMessage   : [2====开始消费REDIS消息队列TOPIC_TEST2数据...],消息数据["222222"]
2020-09-16 01:04:10.344  INFO 9741 --- [    container-5] c.h.r.p.subscribe.ReceiverRedisMessage   : [5====开始消费REDIS消息队列TOPIC_TEST5数据...],消息数据["222222"]
2020-09-16 01:04:10.425  INFO 9741 --- [    container-6] c.h.r.p.subscribe.ReceiverRedisMessage   : [3====开始消费REDIS消息队列TOPIC_TEST3数据...],消息数据[{"@class":"cn.happyloves.redis.TestVO","name":"张三"}]
2020-09-16 01:04:10.425  INFO 9741 --- [    container-7] c.h.r.p.subscribe.ReceiverRedisMessage   : [5====开始消费REDIS消息队列TOPIC_TEST5数据...],消息数据[{"@class":"cn.happyloves.redis.TestVO","name":"张三"}]
2020-09-16 01:04:10.455  INFO 9741 --- [    container-8] c.h.r.p.subscribe.ReceiverRedisMessage   : [4====开始消费REDIS消息队列TOPIC_TEST4数据...],消息数据[{"@class":"cn.happyloves.redis.TestVO","name":"李四"}]
2020-09-16 01:04:10.455  INFO 9741 --- [    container-9] c.h.r.p.subscribe.ReceiverRedisMessage   : [5====开始消费REDIS消息队列TOPIC_TEST5数据...],消息数据[{"@class":"cn.happyloves.redis.TestVO","name":"李四"}]
```
