---
published: true
title: 【Flask数据库】limit、offset以及切片操作
category: flask_MySQL
tags:
  - python
  - flask
  - MySQL
layout: post
---
# 1、limit：可以限制每次查询的时候只查询几条数据。
```python
class Article(Base):
    __tablename__ = 'article'
    id = Column(Integer,primary_key=True,autoincrement=True)
    title = Column(String(50),nullable=False)
    create_time = Column(DateTime,default=datetime.now)

articles = session.query(Article).limit(10).all()
```
# 2、offset：可以限制查找数据的时候过滤掉前面多少条。
```python
articles = session.query(Article).order_by(Article.id.desc()).offset(10).limit(10).all() #从第二十篇开始，往前找10篇
```
# 3、slice切片：可以对Query对象使用切片操作，来获取想要的数据。      
1)`slice(self,start,stop)`start从0开始，含，stop不含。
```python
articles = session.query(Article).order_by(Article.id.desc()).slice(0,10).all()
```
2)**[  ]:常用方法**
```python
articles = session.query(Article).order_by(Article.id.desc())[0:10]
```
