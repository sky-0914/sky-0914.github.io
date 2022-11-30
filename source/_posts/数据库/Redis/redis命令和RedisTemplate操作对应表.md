---
title: redis命令和RedisTemplate操作对应表
tags:
  - Redis
categories:
  - 数据库
---
# redis命令和RedisTemplate操作对应表

+ <a herf="#string">redisTemplate.opsForValue();//操作字符串</a>
+ <a herf="#hash">redisTemplate.opsForHash();//操作hash</a>
+ <a herf="#set">redisTemplate.opsForedisTemplateet();//操作set</a>
+ <a herf="#list">redisTemplate.opsForList();//操作list</a>
+ redisTemplate.opsForZSet();//操作有序set

## Key相关的

| Redis命令                                      | RedisTemplate redisTemplate                                  |                     说明                     |
| :--------------------------------------------- | :----------------------------------------------------------- | :------------------------------------------: |
| `KEYS *、KEYS *key*、KEYS *key、KEYS key*`     | `redisTemplate.keys(key);`                                   |  获取所有key，模糊查询*key*（支持通配符*）   |
| `EXPIRE key 10`<br />`EXPIREAT key 1293840000` | `redisTemplate.expire(key, time, TimeUnit.SECONDS);`<br />`redisTemplate.expireAt(key, date);` | 指定key缓存失效时间<br />指定key缓存到期时间 |
| `TTL key`                                      | `redisTemplate.getExpire(key, TimeUnit.SECONDS);`            |             根据key获取过期时间              |
| `EXISTS key`<br />`EXISTS key1 key2 key3`      | `redisTemplate.hasKey(key);`<br />`redisTemplate.countExistingKeys(Arrays.asList(key));` |    判断key是否存在<br />检查key存在的数量    |
| `DEL key`<br />`DEL key1 key2 key3`            | `redisTemplate.delete(key);`<br />`redisTemplate.delete(Arrays.asList(keys));` |       删除指定key缓存<br />批量删除key       |

<h2 id="string">String相关的</h2>

**redisTemplate.opsForValue();//操作字符串**

| Redis命令                                 | RedisTemplate redisTemplate                                  | 说明                                                         |
| ----------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `SET key value`<br />`SET key value time` | `redisTemplate.opsForValue().set(key,value);`<br />`redisTemplate.opsForValue().set(key,value,time);` | 设置普通缓存<br />设置普通缓存过期时间                       |
| `GET key`                                 | `redisTemplate.opsForValue().get(key);`                      | 获取普通缓存                                                 |
| `INCE key`                                | `redisTemplate.opsForValue().increment(key, delta);`         | 递增                                                         |
| `DECR key`                                | `redisTemplate.opsForValue().increment(key, -delta);`        | 递减                                                         |
| `SETNX key value`                         | `redisTemplate.opsForValue().setIfAbsent(key,value);`        | 将 key 的值设为 value ，当且仅当 key 不存在                  |
| `SETEX key value`                         | `redisTemplate.opsForValue().setIfPresent(key,value);`       | 判断当前的键的值是否为v，是的话不作操作，不实的话进行替换。如果没有这个键也不会做任何操作。 |
| `GETSET key value`                        | `redisTemplate.opsForValue().getAndSet(key, value);`         | key存在设置新值，并返回旧值                                  |

<h2 id="hash">Hash相关的</h2>

**redisTemplate.opsForHash();//操作字符串**

| Redis命令                           | RedisTemplate redisTemplate                             | 说明                                                |
| ----------------------------------- | ------------------------------------------------------- | --------------------------------------------------- |
| `HMSET key key1 value1 key2 value2` | `redisTemplate.opsForHash().putAll(key, map);`          | 设置缓存                                            |
| `HSET key item value`               | `redisTemplate.opsForHash().put(key, item, value);`     | 向一张hash表中放入数据,如果不存在将创建             |
| `HGET key item`                     | `redisTemplate..opsForHash().get(key, item);`           | 获取缓存，字段值                                    |
| `HMGET key`                         | `redisTemplate.opsForHash().entries(key);`              | 获取hashKey对应的所有键值                           |
| `DELETE key item1 item2 item3`      | `redisTemplate.opsForHash().delete(key, item);`         | 将 key 的值设为 value ，当且仅当 key 不存在         |
| `HEXISTS key item`                  | `redisTemplate.opsForHash().hasKey(key, item);`         | 判断hash表中是否有该项的值                          |
| `HINCRBY key item by`               | `redisTemplate.opsForHash().increment(key, item, by);`  | hash递增 如果不存在,就会创建一个 并把新增后的值返回 |
| `HDEL key item by`                  | `redisTemplate.opsForHash().increment(key, item, -by);` | hash递减                                            |

<h2 id="set">Set相关的</h2>

**redisTemplate.opsForedisTemplateet();//操作字符串**

| Redis命令                 | RedisTemplate redisTemplate                                  | 说明                              |
| ------------------------- | ------------------------------------------------------------ | --------------------------------- |
| `SMEMBEredisTemplate key` | `redisTemplate.opsForedisTemplateet().memberedisTemplate(key)` | 根据key获取Set中的所有值          |
| `SISMEMBER key value`     | `redisTemplate.opsForedisTemplateet().isMember(key, value);` | 根据value从一个set中查询,是否存在 |
| `SADD key value1 value2`  | `redisTemplate.opsForedisTemplateet().add(key, values);`     | 将数据放入set缓存                 |
| `SCARD key`               | `redisTemplate.opsForedisTemplateet().size(key);`            | 获取set缓存的长度                 |
| `SREM key value1 value2`  | `redisTemplate.opsForedisTemplateet().remove(key, values);`  | 移除值为value的                   |

<h2 id="list">List相关的</h2>

**redisTemplate.opsForList();//操作字符串**

| Redis命令              | RedisTemplate redisTemplate                             | 说明                         |
| ---------------------- | ------------------------------------------------------- | ---------------------------- |
| `RPUSH key value``     | `redisTemplate.opsForList().rightPush(key, value);`     | 将list放入缓存,从右边添加    |
| `LPUSH key value`      | `redisTemplate.opsForList().leftPush(key, value);`      | 将list放入缓存,从左边添加    |
| `LRANGE key 0 -1`      | `redisTemplate.opsForList().range(key, start, end);`    | 获取list缓存的内容           |
| `LLEN key`             | `redisTemplate.opsForList().size(key);`                 | 获取list缓存的长度           |
| `LINDEX key index`     | `redisTemplate.opsForList().index(key, index);`         | 通过索引 获取list中的值      |
| `LSET key index value` | `redisTemplate.opsForList().set(key, index, value);`    | 根据索引修改list中的某条数据 |
| `LREM key count value` | `redisTemplate.opsForList().remove(key, count, value);` | 移除N个值为value             |

