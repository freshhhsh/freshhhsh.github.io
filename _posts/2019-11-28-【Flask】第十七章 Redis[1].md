---
published: true
title: 【Flask】第十七章 Redis[1]
category: flask
tags:
  - python
  - flask
layout: post
---
# Redis概述和使用场景介绍
## 概述
`redis`是一种`nosql`数据库(无关联数据库),他的数据是保存在内存中，同时`redis`可以定时把内存数据同步到磁盘，即可以将数据持久化，并且他比`memcached`支持更多的数据结构(`string,list列表[队列和栈],set[集合],sorted set[有序集合],hash(hash表)`)。相关参考文档：http://redisdoc.com/index.html
## redis使用场景
1. 登录会话存储：存储在redis中，与memcached相比，数据不会丢失。
2. 排行版/计数器：比如一些秀场类的项目，经常会有一些前多少名的主播排名。还有一些文章阅读量的技术，或者新浪微博的点赞数等。
3. 作为消息队列：比如celery就是使用redis作为中间人。
4. 当前在线人数：还是之前的秀场例子，会显示当前系统有多少在线人数。
5. 一些常用的数据缓存：比如我们的BBS论坛，板块不会经常变化的，但是每次访问首页都要从mysql中获取，可以在redis中缓存起来，不用每次请求数据库。
6. 把前200篇文章缓存或者评论缓存：一般用户浏览网站，只会浏览前面一部分文章或者评论，那么可以把前面200篇文章和对应的评论缓存起来。用户访问超过的，就访问数据库，并且以后文章超过200篇，则把之前的文章删除。
7. 好友关系：微博的好友关系使用redis实现。
8. 发布和订阅功能：可以用来做聊天软件。

## redis和memcached的比较
|   |memcached|redis|
|----|--------|-----|
|类型|纯内存缓存系统|内存磁盘同步数据库|
|数据类型|在定义value时就要固定数据类型|不需要|
|虚拟内存|	不支持|	支持|
|过期策略|	支持|	支持|
|存储数据安全|	不支持|	可以将数据同步到dump.db中|
|灾难恢复|	不支持|	可以将磁盘中的数据恢复到内存中|
|分布式|	支持|	主从同步|
|订阅与发布|	不支持|	支持|

# Redis的安装以及客户端连接
## redis在ubuntu系统中的安装与启动
1. 安装：
 sudo apt-get install redis-server
2. 卸载：
 sudo apt-get purge --auto-remove redis-server
3. 启动：redis安装后，默认会自动启动，可以通过以下命令查看：

 ps aux|grep redis
如果想自己手动启动，可以通过以下命令进行启动：

 sudo service redis-server start
4. 停止：

 sudo service redis-server stop

## 对redis的操作
对redis的操作可以用两种方式，第一种方式采用redis-cli，第二种方式采用编程语言，比如Python、PHP和JAVA等。
1. 使用redis-cli对redis进行字符串操作：

2. 启动redis：
```
sudo service redis-server start
```

3. 连接上redis-server：
```
redis-cli -h [ip] -p [端口]
```

# Redis字符串与过期时间操作
## 添加字符串：
```shell
set key value
如：
set username helloworld
set username2 "hello world"
```
将字符串值value关联到key。如果key已经持有其他值，set命令就覆写旧值，无视其类型。并且默认的过期时间是永久，即永远不会过期。
## 获取值：
```shell
get key
如：
get username
```
## 删除

```shell
del key
如：
del username
```

## 设置过期时间：
若不设置过期时间，则永久不过期
```
expire key timeout(单位为秒)
```
也可以在设置值的时候，一同指定过期时间：
```shell
set key value EX timeout
或：
set key value ex timeout
```
## 查看过期时间：
```shell
ttl key
如：
ttl username
```
## 查看当前redis中的所有key：
```
  keys *
```

# Redis列表操作
## 在列表左边添加元素：
```shell
  lpush key value
```
将值value插入到列表key的表头。如果key不存在，一个空列表会被创建并执行lpush操作。当key存在但不是列表类型时，将返回一个错误。
## 在列表右边添加元素：
```shell
  rpush key value
```
将值value插入到列表key的表尾。如果key不存在，一个空列表会被创建并执行RPUSH操作。当key存在但不是列表类型时，返回一个错误。
## 查看列表中的元素：
```shell
  lrange key start stop
  如：
  lrange usernames 0 -1
```
返回列表key中指定区间内的元素，区间以偏移量start和stop指定,如果要左边的第一个到最后的一个lrange key 0 -1。
## 移除列表中的元素：

### 移除并返回列表key的头元素：
`lpop key`
### 移除并返回列表的尾元素：
`rpop key`
### 移除并返回列表key的中间元素：
`lrem key count value`

将删除key这个列表中，count个值为value的元素。

## 指定返回第几个元素：
`lindex key index`
:将返回key这个列表中，索引为index的这个元素。

## 获取列表中的元素个数：
```shell
  llen key
  如：
  llen languages
```
## 删除指定的元素：
```shell
  lrem key count value
  如：
  lrem languages 0 php
```
根据参数 count 的值，移除列表中与参数 value 相等的元素。count的值可以是以下几种：

* count > 0：从表头开始向表尾搜索，移除与value相等的元素，数量为count。
* count < 0：从表尾开始向表头搜索，移除与 value相等的元素，数量为count的绝对值。
* count = 0：移除表中所有与value 相等的值。
