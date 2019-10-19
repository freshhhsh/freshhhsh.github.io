---
published: true
title: 【Flask数据库】join实现复杂查询
category: flask_MySQL
tags:
  - python
  - flask
  - MySQL
layout: post
---
# 高级查询：
### 1、join方法：
join查询分为两种，一种是inner join，另一种是outer join。默认的是inner join，如果指定left join或者是right join则为outer join。
####内连接：inner join，两张被内连接的表主键不匹配时，不匹配的数据均被抛弃。
####外连接：left join是以左表为基准，连接过程中右表不匹配项会被设置为null，左表数据保留；right join是以右表为基准，连接过程中左表不匹配项会被设置为null，右表数据保留。
* 1)通过原生sql语句实现：
```python
select user.username,count(article.id) from user join article on user.id = article.uid group by user.id order by count(article.id) desc;
从join后的user与article中提取数据，显示不同user的article数量并以降序的方式排列出来。
```
* 2)如果想要查询User及其对应的Article，则可以通过以下方式来实现(默认join为内连接)：
```python
result = session.query(User.username,func.count(Article.id)).join(Article,User.id == Article.uid).group_by(User.id).order_by(func.count(Article.id)).desc()
```
这是通过普通方式的实现，也可以通过join的方式实现，更加简单：
```python
  for u,a in session.query(User,Address).join(Address).all():
    print u
    print a
  # 输出结果：
  >>> <User (id=1,name='ed',fullname='Ed Jason',password='123456')>
  >>> <Address id=4,email=ed@google.com,user_id=1>
```
当然，如果采用outerjoin，可以获取所有user，而不用在乎这个user是否有address对象，并且outerjoin默认为左外查询：
```python
  for instance in session.query(User,Article).outerjoin(Article).all():
    print instance
  # 输出结果：
  (<User (id=1,name='ed')>, <Article id=4,email=ed@google.com,user_id=1>)
  (<User (id=2,name='xt')>, None)
```
