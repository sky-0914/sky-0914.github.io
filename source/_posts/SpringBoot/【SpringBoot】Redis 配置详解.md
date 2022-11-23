---
title: SpringBoot Redis 配置详解
tags:
  - SpringBoot
  - Redis
categories:
  - SpringBoot
---
# SpringBoot Redis 配置详解

```yaml
spring:
  redis:
    database: 1
    host: ${host}
    port: ${port}
    password: ${password}
    timeout: 10000
    lettuce:
      pool:
        max-active: 200 # 连接池最大连接数（使用负值表示没有限制）
        max-wait: -1ms # 连接池最大阻塞等待时间（使用负值表示没有限制）
        max-idle: 200 # 连接池中的最大空闲连接
        min-idle: 50 # 连接池中的最小空闲连接
        #驱逐时间 初始化延迟时间 默认-1
        #if (delay > 0L) 必须>时才初始化
        #(这句很重要，这个参数默认为-1,不>0L则驱逐任务不会生成，池中的idel max 等参数等于没配置，不会生效。第二，如果是默认配置，即使生效也不会读池中链接，更浪费资源！！！代码见下边引用)
        time-between-eviction-runs: 1s
```

[原文连接](https://www.jianshu.com/p/ac6abfaeebbc?from=groupmessage)
