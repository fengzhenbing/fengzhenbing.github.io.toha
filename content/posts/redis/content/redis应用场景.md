---
title: "redis应用场景"
date: 2020-12-13T08:06:25+06:00
description: redis应用场景
menu:
  sidebar:
    parent: redis
    name: redis应用场景
    identifier: redis3
    weight: 3
---

### 一.业务数据缓存

经典用法。

1.  通用数据缓存，string，int，list，map等。
2.  实时热数据，最新500条数据。 
3. 会话缓存，token缓存等。

### 二.业务数据处理

1. 非严格一致性要求的数据:评论，点击等。 
2. 业务数据去重:订单处理的幂等校验等。 
3. 业务数据排序:排名，排行榜等。

### 三.全局一致计数 

1. 全局流控计数 
2. 秒杀的库存计算 
3. 抢红包 
4. 全局ID生成

### 四.高效统计计数

1. id去重，记录访问ip等全局bitmap操作 
2. UV、PV等访问量==>非严格一致性要求

### 五.发布订阅与Stream

1. Pub-Sub 模拟队列

2. Redis Stream 是 Redis 5.0 版本新增加的数据结构。
   Redis Stream 主要用于消息队列(MQ，Message Queue)。 

   具体可以参考 https://www.runoob.com/redis/redis-stream.html

### 六.分布式锁

1、获取锁--单个原子性操作

 ```
SET dlock my_random_value NX PX 30000
 ```

2、释放锁--lua脚本-保证原子性+单线程，从而具有事务性 

```
if redis.call("get",KEYS[1]) == ARGV[1] then
return redis.call("del",KEYS[1]) else
return 0 end
```

- 关键点:原子性、互斥、超时