---
published: true
title: 【Flask】第十六章 memcached
category: flask
tags:
  - python
  - flask
layout: post
---
# memcached介绍
1. memcached之前是danga的一个项目，最早是为LiveJournal服务的，当初设计师为了加速LiveJournal访问速度而开发的，后来被很多大型项目采用。官网是www.danga.com 或者是 memcached.org。
2. Memcached是一个高性能的**分布式**的**内存对象缓存系统**，全世界有不少公司采用这个缓存项目来构建大负载的网站，来分担数据库的压力。Memcached是通过在内存里维护一个统一的巨大的hash表，memcached能存储各种各样的数据，包括图像、视频、文件、以及数据库检索的结果等。简单的说就是将数据调用到内存中，然后从内存中读取，从而大大提高读取速度。
3. 哪些情况下适合使用Memcached：存储验证码（图形验证码、短信验证码）、登录session等所有不是至关重要的数据。

# memcached的安装与参数详解
## memcached的安装
1. windows：
安装：`memcached.exe -d install`
启动：`memcached.exe -d start`
2. linux（ubuntu）：
官网下载ubantu服务器版，使用Oracle VM创建虚拟机并安装系统。之后在ubantu中：
* 安装：`sudo apt install memcached`
* 启动：`/usr/bin/memcached -u memcache -d start`
3. 可能出现的问题：
提示你没有权限：在打开`cmd`的时候，右键使用管理员身份运行。
提示缺少`pthreadGC2.dll`文件：将`pthreadGC2.dll`文件拷贝到`windows/System32`.
不要放在含有中文的路径下面。

## 启动memcached与参数详解
-d：这个参数是让`memcached`在后台运行。
-m：指定占用多少内存。以M为单位，默认为64M。`-m 1024`
-p：指定占用的端口。默认端口是11211。`-p 11222`
-l：别的机器可以通过哪个ip地址连接到我这台服务器。

如果是通过`service memcached start`的方式，那么只能通过本机连接。如果想要让别的机器连接，就必须设置`-l 0.0.0.0`。
如果想要使用以上参数来指定一些配置信息，那么不能使用`service memcached start`，而应该使用`/usr/bin/memcached`的方式来运行。比如`/usr/bin/memcached -u memcache -m 1024 -p 11222 start`。

# telnet操作memcached
命令提示符输入：
`telnet ip地址 端口号[默认11211]`，如`telnet 127.0.0.1 11211`
## 退出
`>quit`
## 添加数据(更改数据)：

### set：如果存在，则更改数据的值
* 语法：
```shell
>set key 0/1(是否压缩) timeout value_length
>value
```
* 示例：
```shell
>set username 0 60 7
>zhiliao
```
### add:如果存在，则添加失败

* 语法：
```shell
>add key 0/1(是否压缩) timeout value_length
>value
```
* 示例：
```shell
add username 0 60 7
xiaotuo
```

### incr:增加数字类型数据的值
* 语法：
```shell
>incr key value
```
* 示例：
```shell
>set price 0 120 2
>20
>incr price 5
>get price
(结果25)
```

### decr:减少数字类型数据的值
* 语法：
```shell
>decr key value
```
* 示例：
```shell
>set price 0 120 2
>20
>decr price 5
>get price
(结果15)
```

## 获取数据
### get:
* 语法：
```shell
>get key
```
示例：
```shell
>get username
```

## 删除数据
### delete：删除单条数据
* 语法：
```shell
>delete key
```
* 示例：
```shell
>delete username
```
### flush_all：删除memcached中的所有数据
```shell
>flush_all
```

## 查看状态
查看memcached的当前状态：
语法：`>stats`。

# python操作memcached
## 安装
`pip install python-memcached`
## 建立连接
连接前一定要先启动memcache。可指定多个分布式服务器IP地址，以做成分布式服务器，之后memcached会自动分配存储数据。
```python
import memcached
mc = memcache.Client(['127.0.0.1:11211','192.168.174.130:11211'],debug=True)
```
## 设置数据
```py
#单条数据
mc.set('username','hello world',time=60*5) #如果不设置过期时间，则会默认为0，即永远存在
#多条数据
mc.set_multi({'email':'xxx@qq.com','telphone':'111111'},time=60*5)
```
## 获取数据
`result = mc.get('telphone')`
## 删除数据
`mc.delete('email')`
## 自增长
```py
mc.incr('read_count',delta = 10) #若不传入增长步长，则默认为1
mc.incr('read_count')
```
## 自减少
```python
mc.decr('read_count',delta = 10) #若不传入减少步长，则默认为1
```

# memcached的安全性
`memcached`的操作不需要任何用户名和密码，只需要知道`memcached`服务器的`ip`地址和端口号即可。因此`memcached`使用的时候尤其要注意他的安全性。这里提供两种安全的解决方案。分别来进行讲解：

* 使用`-l`参数设置为只有本地可以连接，即默认不设置参数`-l`：这种方式，就只能通过本机才能连接，别的机器都不能访问，可以达到最好的安全性。

* 使用防火墙，关闭`11211`端口，外面也不能访问。
```shell
ufw -h #命令帮助
ufw enable # 开启防火墙
ufw disable # 关闭防火墙
ufw default deny # 防火墙以禁止的方式打开，默认是关闭那些没有开启的端口
ufw deny 端口号 # 关闭某个端口
ufw allow 端口号 # 开启某个端口
```
