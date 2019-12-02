---
published: true
title: 【Flask】第十七章 Redis[3]
category: flask
tags:
  - python
  - flask
layout: post
---
# Redis设置连接密码
## 修改配置文件启用密码登陆
```shell
cd /etc/redis
vim redis.conf
```
进入vim编辑后：
```shell
/requirepass #搜索requirepass,alt然后按n查找
#将requirepass password的注释取消掉，然后替换password为指定密码并保存
```
## 使用密码登陆redis
打开redis并登陆进入：

### 方法一
```shell
redis-cli -p 6379 -h 127.0.0.1
auth [password]   #授权之后便可继续操作redis
```
### 方法二
```shell
redis-cli -p 6379 -h 127.0.0.1 -a [password]
```

# 其他机器连接本机Redis
```shell
cd /etc/redis/
vim redis.conf
```

```shell
/bind #搜索bind并以此下形式修改：
#bind ip1 ip2 ip3，如：
bind 127.0.0.1 192.168.1.102
!!!!!注意!!!!!
127.0.0.1是本机连接redis时的ip
192.168.1.102是其他机器连接本机时本机的ip地址！
```

# Python操作Redis
## 安装python-redis：
`pip install redis`
## 具体操作
新建一个文件比如redis_test.py，然后初始化一个redis实例变量，并且在ubuntu虚拟机中开启redis。比如虚拟机的ip地址为192.168.174.130。示例代码如下：
```py
# 从redis包中导入Redis类
from redis import Redis
# 初始化redis实例变量
cache = Redis(host='192.168.1.15',port=6379,password='zhiliao')
```
### 对字符串的操作：
操作redis的方法名称，跟之前使用redis-cli一样，现就一些常用的来做个简单介绍，示例代码如下(承接以上的代码)：
```py
# 添加一个值进去，并且设置过期时间为60秒，如果不设置，则永远不会过期
cache.set('username','xiaotuo',ex=60)
# 获取一个值
cache.get('username')
# 删除一个值
cache.delete('username')
# 给某个值赋值1
cache.set('read_count',1)
# 给某个值自增1
cache.incr('read_count')  # 这时候read_count变为2
# 给某个值减少1
cache.decr('read_count') # 这时候read_count变为1
```
### 对列表的操作：
同字符串操作，所有方法的名称跟使用redis-cli操作是一样的：
```py
# 给languages这个列表往左边添加一个python
cache.lpush('languages','python')
# 给languages这个列表往左边添加一个php
cache.lpush('languages','php')
# 给languages这个列表往左边添加一个javascript
cache.lpush('languages','javascript')

# 获取languages这个列表中的所有值
print(cache.lrange('languages',0,-1))
> ['javascript','php','python']
```

### 对集合的操作：
```py
# 给集合team添加一个元素xiaotuo
cache.sadd('team','xiaotuo')
# 给集合team添加一个元素datuo
cache.sadd('team','datuo')
# 给集合team添加一个元素slice
cache.sadd('team','slice')

# 获取集合中的所有元素
xtredis.smembers('team')
> ['datuo','xiaotuo','slice'] # 无序的
```

### 对哈希(hash)的操作：
```py
# 给website这个哈希中添加baidu
cache.hset('website','baidu','baidu.com')
# 给website这个哈希中添加google
cache.hset('website','google','google.com')

# 获取website这个哈希中的所有值
print(cache.hgetall('website'))
> {"baidu":"baidu.com","google":"google.com"}
```

### 事务(管道)操作：
redis支持事务操作，也即一些操作只有统一完成，才能算完成。否则都执行失败，用python操作redis也是非常简单，示例代码如下：
```py
# 定义一个管道实例
pip = cache.pipeline()
pip.set('BankA',1)
pip.set('BankB',2)
# 做第一步操作，给BankA自增长1
pip.incr('BankA')
# 做第二步操作，给BankB自减少1
pip.desc('BankB')
# 执行事务
pip.execute()
```

### 发布与订阅功能
异步发送邮件功能
main.py：负责监听
```py
ps = cache.pubsub()
ps.subscrib('email')  #订阅email频道
while True:
    for item in ps.listen():
        print(item)
        if item['type'] == 'message':
            data = item['data']  #获取频道发布的内容
            print(data)
            # 再发送邮件，省略
```
publish_email.py：负责发布
```py
from redis import Redis

cache = Redis(host='192.168.1.15',port=6379,password='zhiliao')

for x in range(3):
    cache.publish('email','xxx@qq.com')  #在email频道发布消息‘xxx@qq.com’
```
