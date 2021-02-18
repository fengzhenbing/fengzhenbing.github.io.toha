---
title: "redis数据结构"
date: 2020-12-11T08:06:25+06:00
description: redis数据结构
menu:
  sidebar:
    parent: redis
    name: redis数据结构
    identifier: redis2
    weight: 2
---

### redis-benchmark

- 环境mac 4核8g

```shell
mokernetdeMac-mini:redis-6.0.9 mokernet$ ./bin/redis-benchmark -n 100000 -c 32 -t SET,GET,INCR,HSET,LPUSH,MSET -q
SET: 121065.38 requests per second
GET: 118764.84 requests per second
INCR: 117508.81 requests per second
LPUSH: 123001.23 requests per second
HSET: 123915.74 requests per second
MSET (10 keys): 96711.80 requests per second
```

### redis的5种基本数据结构

https://redis.io/commands

#### 1.字符串(string)

简单来说就是三种:int、string、byte[]

Redis中字符串类型的value最多可以容纳的数据长度是512M

```
set/get/getset/del/exists/append incr/decr/incrby/decrby
```



#### 2.散列(hash)

Redis中的Hash类型可以看成具有String key 和String value的map容器。

```
hset/hget/hmset/hmget/hgetall/hdel/hincrby hexists/hlen/hkeys/hvals
```



* hmset 相对于hset可一次设置多个键值对
* hmget 相对于hget可一次获取多个键的值

####  3.列表(list)

java的LinkedList

在Redis中，List类型是按照插入顺序排序的字符串链表。和数据结构中的普通链表 一 样，我们可以在其头部(Left)和尾部(Right)添加新的元素。在插入时，如果该键并不存 在，Redis将为该键创建一个新的链表。与此相反，如果链表中所有的元素均被移除， 那么该键也将会被从数据库中删除。

```
lpush/rpush/lrange/lpop/rpop
```



#### 4.集合(set)

java的set，不重复的list

在redis中，可以将Set类型看作是没有排序的字符集合，和List类型一样，我们也可以 在该类型的数值上执行添加、删除和判断某一元素是否存在等操作。这些操作的时间复 杂度为O(1),即常量时间内完成依次操作。

和List类型不同的是，Set集合中不允许出现重复的元素。

```
sadd/srem/smembers/sismember  类比java中set的add, remove, all,  contains, 
sdiff/sinter/sunion  集合求差集，求交集，求并集
```



#### 5.有序集合(sorted set)

按权重排序

sortedset在set基础上给每个元素加了个分数score。

redis 正是通过分数来为集合的成员进行从小到大的排序。sortedset中分数是可以重复的。

```
zadd key score member score2 member2... : 将成员以及该成员的分数存放到sortedset中 
zscore key member : 返回指定成员的分数
zcard key : 获取集合中成员数量
zrem key member [member...] : 移除集合中指定的成员，可以指定多个成员
zrange key start end [withscores] : 获取集合中脚注为start-end的成员，[withscores]参数表明返回的成员 包含其分数
zrevrange key start stop [withscores] : 按照分数从大到小的顺序返回索引从start到stop之间的所有元素 (包含两端的元素)
zremrangebyrank key start stop : 按照排名范围删除元素
```

### redis高级数据结构

#### Bitmaps

bitmaps不是一个真实的数据结构。而是String类型上的一组面向bit操作的集合。由于 strings是二进制安全的blob，并且它们的最大长度是512m，所以bitmaps能最大设置 2^32个不同的bit。

```
setbit/getbit/bitop/bitcount/bitpos
```

可用作集中式的冥等去重    数据压缩在字节位上，极大的节约了空间



#### HyperLogLog

Redis  是用来做基数统计的算法，HyperLogLog 的优点是，在输入元素的数量或者体积非常非常大时，计算基数所需的空间总是固定 的、并且是很小的。

在 Redis 里面，每个 HyperLogLog 键只需要花费 12 KB 内存，就可以计算接近 2^64 个不同元素的基 数。这和计算基数时，元素越多耗费内存就越多的集合形成鲜明对比。

但是，因为 HyperLogLog 只会根据输入元素来计算基数，而不会储存输入元素本身，所以 HyperLogLog 不能像集合那样，返回输入的各个元素。

```
pfadd 添加
pfcount 获得基数值
pfmerge 合并多个key
```

```shell
127.0.0.1:6379> pfadd pf1 1 3 1 4 1 5
(integer) 1
127.0.0.1:6379> pfcount pf1
(integer) 4
127.0.0.1:6379> pfadd pf2 3 4 5  6 7
(integer) 1
127.0.0.1:6379> pfcount pf2
(integer) 5
127.0.0.1:6379> pfmerge pf3 pf1 pf2
OK
127.0.0.1:6379> pfcount pf3
(integer) 6
```



应用场景

说明：

- 基数不大，数据量不大就用不上，会有点大材小用浪费空间
- 有局限性，就是只能统计基数数量，而没办法去知道具体的内容是什么
- 和bitmap相比，属于两种特定统计情况，简单来说，HyperLogLog 去重比 bitmap 方便很多
- 一般可以bitmap和hyperloglog配合使用，bitmap标识哪些用户活跃，hyperloglog计数

一般使用：

- 统计注册 IP 数
- 统计每日访问 IP 数
- 统计页面实时 UV 数
- 统计在线用户数
- 统计用户每天搜索不同词条的个数



#### GEO

```
geoadd/geohash/geopos/geodist/georadius/georadiusbymember
```



Redis的GEO特性在 Redis3.2版本中推出，这个功能可以将用户给定的地理位置(经 度和纬度)信息储存起来，并对这些信息进行操作。