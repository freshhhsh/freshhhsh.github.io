---
published: true
title: 【Flask数据库】Column常用参数
category: flask_数据库
tags:
  - python
  - flask
  - MySQL
layout: post
---
@[TOC](文章目录)
#一、Column常用参数：

```
class Article(Base):
    __tablename__ = 'article'
    id = Column(Integer,primary_key = True,autoincrement=True)
    read_count = Column(Integer, default = 11)
    create_time = Column(Datetime, default = datetime.now)  #注，now方法不需要执行，会在提交时自动执行

Base.metadata.drop_all()
Base.metadata.create_all()

```
****
* default：默认值
如添加时不指定read_count的default默认值,则默认为null
****
* nullable: 是否为空,True/False
默认nullable为True，若`read_count = Column(Integer,nullable = False)`,则不指定read_count值时添加数据会报错。
****
* primary_key：是否为主键，True/False
****
* unique：是否唯一，True/False
默认为False，若`read_count = Column(Integer,unique= True)`,则read_count值与表内某条数据的值重复时添加数据会报错。
****
* autoincrement：是否自增长。
****
* onupdate：更新该条数据的时候自动执行
如：
`update_time = Column(DateTime,onupdate=datetime.now,default = datetime.now)`
注：在第一次创建数据的时候，不会使用onupdate的值，而会使用default值
```
article = session.query(Article).first()
article.title = 'new title'
session.commit()
```
****
* name：该属性在数据库中的字段名
当作关键字参数：
`update_time = Column(DateTime,name = '更新时间')`
或当作位置参数，在第一个位置：
`update_time = Column('更新时间',DateTime)`
