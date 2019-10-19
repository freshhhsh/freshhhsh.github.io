---
published: true
title: 【Flask数据库】query函数可查询的数据
category: flask_MySQL
tags:
  - python
  - flask
  - MySQL
layout: post
---
# 一、query可用参数：
### 1、模型对象。指定查找这个模型中的所有对象
```
articles = session.query(Article).all()
for article in articles:
    print(article)#重新实现__str__方法
```
### 2、模型中的属性。可以指定只查找某个模型的其中几个属性。
```
articles = session.query(Article.title,Article.price).all()#返回列表，其内元素为二元元组
print(articles)#因articles是列表，重新实现__repr__方法
```
### 3、聚合函数
需`from sqlalchemy import func`
* func.count：统计行的数量。
`session.query(func.count(Article.id)).first()`
* func.avg：求平均值。
`session.query(func.avg(Article.price)).first()`
* func.max:求最大值。
略
* func.min：求最小值。
略
* func.sum: 求和。
*****
注：实际上，只要MySQL上有的函数，都可用func.函数名来调用。
