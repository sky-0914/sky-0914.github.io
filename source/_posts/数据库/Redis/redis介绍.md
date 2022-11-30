---
title: Redis介绍
tags:
  - Redis
categories:
  - 数据库
---
# 安装Redis
## 搜狐开源项目CacheCloud：https://github.com/sohutv/cachecloud

Redis是IO多路复用，Redis用来接收消息的用的是多线程，但当真正处理请求的时候用的是单线程。

Redis与Memacache区别
Redis处理请求是单线程
Memacache是多线程

Redis支持五种数据结构：String，Hash，Set，ZSet，List
Memacache仅支持一种：String

Redis可以设置持久化到硬盘上
Memacache只在内存中，服务器宕机后重启数据就丢失了

Redis 如何持久化
>RDB持久化配置
>
>默认是RDB也就是快照的意思。Redis默认提供了一种策略，也就是在多少秒内操作了多少次就进行一次快照
>> save 900 1           #在900秒(15分钟)之后，如果至少有1个key发生变化，则dump内存快照。
>
>> save 300 10          #在300秒(5分钟)之后，如果至少有10个key发生变化，则dump内存快照。
>
>> save 60 10000        #在60秒(1分钟)之后，如果至少有10000个key发生变化，则dump内存快照。
>
> AOF持久化配置
>
>> appendfsync always     #每次有数据修改发生时都会写入AOF文件。
> 
>> appendfsync everysec  #每秒钟同步一次，该策略为AOF的缺省策略。
>
>> appendfsync no        #从不同步。高效但是数据不会被持久化。



