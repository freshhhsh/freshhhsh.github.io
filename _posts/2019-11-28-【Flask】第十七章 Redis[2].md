---
published: true
title: 【Flask】第十七章 Redis[2]
category: flask
tags:
  - python
  - flask
layout: post
---
# set集合操作
## 添加元素：
```
sadd set value1 value2....
如：
sadd team xiaotuo datuo
```
## 查看元素：
```
  smembers set
  如：
  smembers team
```
## 移除元素：
```
  srem set member...
  如：
  srem team xiaotuo datuo
```
## 查看集合中的元素个数：
```
  scard set
  如：
  scard team1
```
## 获取多个集合的交集：
```
  sinter set1 set2
  如：
  sinter team1 team2
```
## 获取多个集合的并集：
```
  sunion set1 set2
  如：
  sunion team1 team2
```
## 获取多个集合的差集：
```
sdiff set1 set2
如：
sdiff team1 team2
```
# Redis哈希操作
## 添加一个新值：
```
  hset key field value
  如：
  hset website baidu baidu.com
```
将哈希表key中的域field的值设为value。
如果key不存在，一个新的哈希表被创建并进行 HSET操作。如果域 field已经存在于哈希表中，旧值将被覆盖。
## 获取哈希中的field对应的值：
```
  hget key field
  如：
  hget website baidu
```
## 删除field中的某个field：
```
  hdel key field
  如：
  hdel website baidu
```
## 获取某个哈希中所有的field和value：
```
  hgetall key
  如：
  hgetall website
```
## 获取某个哈希中所有的field：
```
  hkeys key
  如：
  hkeys website
```
## 获取某个哈希中所有的值：
```
hvals key
如：
hvals website
```
## 判断哈希中是否存在某个field：
```
hexists key field
如：
hexists website baidu
```
## 获取哈希中总共的键值对：
```
hlen field
如：
hlen website
```
# Redis的事务操作
## 事务操作特征
Redis事务可以一次执行多个命令，事务具有以下特征：

* 隔离操作：事务中的所有命令都会序列化、按顺序地执行，不会被其他命令打扰。
* 原子操作：事务中的命令要么全部被执行，要么全部都不执行。

## 开启一个事务：
`multi`:以后执行的所有命令，都在这个事务中执行的。如果在执行命令过程中报错，则事务取消。
## 执行事务：
```
exec
```
会将在multi和exec中的操作一并提交。

## 取消事务：
```
discard
```
会将multi后的所有命令取消。

## 监视一个或者多个key：
```
watch key...
```
监视一个(或多个)key，如果在事务执行之前这个(或这些) key被其他命令所改动，那么事务将被打断，即不会再被正常执行。

## 取消所有key的监视：
```
  unwatch
```

# Redis的发布和订阅操作
订阅频道后会自动接收该频道发布的消息。频道无需创建，发布消息或订阅时自动创建。
## 给某个频道发布消息：
```
publish channel message
如：
publish chatroom1 "hello world"
```
## 订阅某个频道的消息：
```
subscribe channel
如
subscribe chatroom1 chatroom2
```

# Redis的RDB和AOF的两种数据持久化机制
`redis`提供了两种数据备份方式，一种是`RDB`，另外一种是`AOF`，以下将详细介绍这两种备份策略：
## 两种方式比较
|   |RDB|AOF|
|----|----|----|
|开启关闭|开启：默认开启。关闭：把配置文件中所有的save都注释，就是关闭了。|开启：在配置文件中appendonly yes即开启了aof，no为关闭。|
|同步机制|可以指定某个时间内发生多少个命令进行同步。比如1分钟内发生了2次命令，就做一次同步。如：(1):  save 900 1：如果在900s以内发生了1次数据更新操作，那么就会做一次同步操作。(2):  save 300 10：如果在300s以内发生了10数据更新操作，那么就会做一次同步操作。(3):  save 60 10000：如果在60s以内发生了10000数据更新操作，那么就会做一次同步操作。|每秒同步或者每次发生命令后同步|
|存储内容|存储的是redis里面的具体的值 | 存储的是执行的更新数据的操作命令
| 存储文件的路径 | 根据dir以及dbfilename来指定路径和具体的文件名 | 根据dir以及appendfilename来指定具体的路径和文件名 |
| 优点 | （1）存储数据到文件中会进行压缩，文件体积比aof小。（2）因为存储的是redis具体的值，并且会经过压缩，因此在恢复的时候速度比AOF快。（3）非常适用于备份。 | （1）AOF的策略是每秒钟或者每次发生写操作的时候都会同步，因此即使服务器故障，最多只会丢失1秒的数据。 （2）AOF存储的是Redis命令，并且是直接追加到aof文件后面，因此每次备份的时候只要添加新的数据进去就可以了。（3）如果AOF文件比较大了，那么Redis会进行重写，只保留最小的命令集合。 |
 | 缺点 | （1）RDB在多少时间内发生了多少写操作的时候就会出发同步机制，因为采用压缩机制，RDB在同步的时候都重新保存整个Redis中的数据，因此你一般会设置在最少5分钟才保存一次数据。在这种情况下，一旦服务器故障，会造成5分钟的数据丢失。（2）在数据保存进RDB的时候，Redis会fork出一个子进程用来同步，在数据量比较大的时候，可能会非常耗时。 | （1）AOF文件因为没有压缩，因此体积比RDB大。 （2）AOF是在每秒或者每次写操作都进行备份，因此如果并发量比较大，效率可能有点慢。（3）AOF文件因为存储的是命令，因此在灾难恢复的时候Redis会重新运行AOF中的命令，速度不及RDB。 |
  | 更多 | http://redisdoc.com/topic/persistence.html#redis |
