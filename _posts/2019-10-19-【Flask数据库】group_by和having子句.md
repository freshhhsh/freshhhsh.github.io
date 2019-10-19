---
published: true
title: 【Flask数据库】group_by和having子句
category: flask_MySQL
tags:
  - python
  - flask
  - MySQL
layout: post
---
# 高级查询：
### 1、group_by
根据某个字段进行分组。比如想要根据性别进行分组，来统计每个分组分别有多少人，那么可以使用以下代码来完成：
```
session.query(User.gender,func.count(User.id)).group_by(User.gender).all()
```
### 2、having
having是对查找结果进一步过滤。比如只想要看未成年人的数量，那么可以首先对年龄进行分组统计人数，然后再对分组进行having过滤。示例代码如下：
```
result = session.query(User.age,func.count(User.id)).group_by(User.age).having(User.age >= 18).all()
```
