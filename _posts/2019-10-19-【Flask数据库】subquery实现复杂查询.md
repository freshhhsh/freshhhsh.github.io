---
published: true
title: 【Flask数据库】subquery实现复杂查询
category: flask_MySQL
tags:
  - python
  - flask
  - MySQL
layout: post
---
```python
class User(Base):
    __tablename__ = 'user'
    id = Column(Integer,primary_key = True,autoincrement=True)
    username = Column(String(50),nullable=False)
    city = Column(String(50),nullable=False)
    age = Column(Integer,default = 0)

    def __repr__(self):
        return "<User(username:  %s)>"% self.username

Base.metadata.drop_all()
Base.metadata.create_all()

user1 = User(username="A",city="changsha",age=18)
user2 = User(username="B",city="changsha",age=18)
user3 = User(username="C",city="beijing",age=18)
user4 = User(username="D",city="changsha",age=20)

session.add_all([user1,user2,user3,user4])
session.commit()
```
需求：寻找和A在同一个城市并且是同年龄的人。
### 1、query传统方式分两次查找：
```python
user = session.query(User).filter(User.username = "A").first()
users = session.query(User).filter(User.city == user.city,User.age = user.age).all()
```
### 2、sql原生传统方式子查找：
```python
select user.id,user.username,user.city,user.age from user,(select user.city,user.age from user where user.username="A") as li_a where user.age = li_a.age and user.city=li_a.city
```
### 3、query子查寻方式：
```python
stmt = session.query(User.city.lable("city"),User.age.lable("age")).filter(User.username=="A").subquery()
session.query(User).filter(User.city ==stmt.c.city,User.age==stmt.c.age).all()
```
调用`subquery()`方法后会返回子查询对象，用lable生成对应属性的别名方便以后调用(`age`和`city`属性)，用`stmt`变量接收后，可以调用`.c`方法，`c`代表`Column`列，`stmt.c.city`返回子查询对象的city值。
