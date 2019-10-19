---
published: true
title: 【Flask数据库】relationship方法中的cascade参数详解(1-2)
category: flask_MySQL
tags:
  - python
  - flask
  - MySQL
layout: post
---
# 一、ORM层面的CASCADE
在SALAlchemy中，只要将一条数据添加到session中，和他相关联的数据都可以一起存入。这其实是通过relationship对象创建时传入关键字参数cascade实现的，另外，也可在backref中传入casecade关键字：
#### 1.`save-update`:
默认选项，自动把其他关联数据添加到数据库中就是这条实现的。若不想同步添加，令`cascade=''`
#### 2.`delete`：
表示当删除某一个模型中的数据的时候，是否也删掉使用`relationship`和他关联的数据。
#### 3.`delete-orphan`：
表示当对一个ORM对象解除了父表中的关联对象的时候，自己便会被删除掉(即子表中的某条数据没有了关联数据，则该条数据会被自动删除掉)。当然如果父表中的数据被删除，自己也会被删除。这个选项只能用在一对多上，不能用在多对多以及多对一上。并且还需要在子模型中的`relationship`中，增加一个`single_parent=True`的参数。
#### 4.`merge`：
默认选项。当在使用`session.merge`，合并一个对象的时候，会将使用了`relationship`相关联的对象也进行`merge`操作。如user表中已有一条数据id=1，而脚本中另外生成了一条id=1的数据，那么就可用merge覆盖掉原数据及相关联数据。实际项目使用较少。
#### 5.`expunge`：
移除操作的时候，会将相关联的对象也进行移除。这个操作与session.delete(obj)不同，它只是从`session`中移除，并等到`session.commit()`时不会真正的从数据库中删除。实际项目使用较少。
#### 6.`all`：
是对`save-update, merge, refresh-expire(极少用), expunge, delete`几种的缩写。
************
# 二、save-update及delete例子

```python
class User(Base):
    __tablename__ = 'user'
    id = Column(Integer,primary_key=True,autoincrement=True)
    username= Column(String(50),nullable=False)

    def __repr__(self):
        return "<Tag(title:%s)>" % self.title

class Article(Base):
    __tablename__ = 'article'
    id = Column(Integer,primary_key=True,autoincrement=True)
    title= Column(String(50),nullable=False)
    uid = Column(Integer,ForeignKey("User.id"))
    author = relationship("User",backref = backref("articles",cascade="save-update,delete"),cascade="save-update")
    #author = relationship("User",backref = "articles"，cascade="save-update,delete")，删除article数据时也会删除user数据


    def __repr__(self):
        return "<Article(title:%s)>" % self.title
############################
Base.metadata.drop_all()
Base.metadata.create_all()

user = User(username="zhiliao")
article = Article(title = "article1")
article.author= user
session.add(article)#cascade=''，所以不会添加article的user属性，即uid
session.commit()
############################
Base.metadata.drop_all()
Base.metadata.create_all()

user = User(username="zhiliao")
article = Article(title = "article1")
article.author= user
session.add(article)#cascade=''，所以不会添加article的user属性，即uid
session.commit()
```
#### 1、delete-orphan
```
class User(Base):
    __tablename__ = 'user'
    id = Column(Integer,primary_key=True,autoincrement=True)
    username= Column(String(50),nullable=False)

    def __repr__(self):
        return "<Tag(title:%s)>" % self.title

class Article(Base):
    __tablename__ = 'article'
    id = Column(Integer,primary_key=True,autoincrement=True)
    title= Column(String(50),nullable=False)
    uid = Column(Integer,ForeignKey("User.id"))
    author = relationship("User",backref = backref("articles",cascade="save-update,delete,delete-orphan"),cascade="save-update",single_parent = True)

    def __repr__(self):
        return "<Article(title:%s)>" % self.title

```
#### 2、merge合并操作及expunge移除操作因不常用，略。
