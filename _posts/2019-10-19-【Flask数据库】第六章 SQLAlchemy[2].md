---
published: true
title: 【Flask】第六章 SQLAlchemy[2]
category: flask_MySQL
tags:
  - python
  - flask
  - MySQL
layout: post
---
# 多对一、一对一关系实现
## 多表同时添加数据
### 多对一关系同时添加实现
问题描述：以user表和article表为例，先需添加一个作者和此作者的几篇文章，需要在添加作者的同时利用外键特性也将作者的几篇文章添加进去。则可在`session.add(user)`之前先把特性绑定在要添加的`user`上，如`user.articles.append(article1)`，再`session.add(user)`。
```python
from sqlalchemy.orm import relationship
class User(Base):
    ...
    username = Column(...)
    #articles = relationship("article")

class Article(Base):
    ...
    author = relationship("User",backref = "articles")

user = User(username='zhiliao')
article1 = Article(title='abc1',content = '123')
article2 = Article(title='dedd',content='yyyy')

user.articles.append(article1)
user.articles.append(article2)
session.add(user)
session.commit()
```
### 一对一关系同时添加实现
问题描述：以user表和article表为例，先需添加一篇文章和此文章的作者，需要在添加文章的同时利用外键特性也将作者添加进去。则可在`session.add(article1)`之前先把特性绑定在要添加的`article1`上，如`article1.author.append(user)`，再`session.add(article1)`。
****
## 一对一关系：表的扩展
问题描述：由于user表中部分栏的数据不经常需要，为节省资源，不需每次查询时都提取其中数据，因此将user表中不常用栏的数据放在扩展表中，取名为`userextend`，并与user表建立一对一关系。即userextend表中一条数据只能对应user表中的单条数据，换句话说，一个`User`对象只能对应一个`UserExtend`对象。需要在创建`relationship`对象时传入`uselist = False`实现。
```python
from sqlalchemy.orm import relationship,backref
class User(Base):
    ...
    username = Column(...)
    #extend = relationship("UserExtend",uselist = False)  

class UserExtend(Base):
    id = Column(Integer, primary_key = True, autoincrement = True)
    school = Column(String(50))
    uid = Column(Integer,ForeignKey("user.id")
    user = relationship("User",backref = backref("extend",uselist = False)) #此条使user.extend属性不再是列表，也不可同时添加多条UserExtend对象.
#因不可再使用append方法，添加单条user的同时添加绑定的userextend数据：
user =User(username = 'zhiliao')
userextend1 = UserExtend(school = 'zhiliao ketang'
user.extend = userextend1
session.add(user)
session.commit()
```

# 多对多关系实现
## 多对多关系
问题说明：一篇文章可以有多个标签，一个标签下也可以有多个文章。
解决方法：构件一张中间表，以存储他们的对应关系。
`from sqlalchemy import Table`
创建一个Table对象并在Article类或Tag类内建立relationship时传入secondary = Table对象。
即：`article_tag =Table(...)`,
`tags = relationship("Tag",backref="articles",secondary = article_tag )`
## 代码实现
```python
from sqlalchemy import Table
article_tag = Table("article_tag",#指定表名
    #在创建表时指定用Base.metadata创建
    Base.metadata,
    #创建名为"article_id"的列，一头连接article.id
    Column("article_id",Integer,ForeignKey("article.id"),primary_key = True),
    #创建名为"tag_id"的列，一头连接tag.id，并且也设置为主键，从而与article_id组成复合主键，复合主键会组合查重，如1-1与1-1重复，而1-1与1-2不重复。这就避免了数据重复问题。
    Column("tag_id",Integer,ForeignKey("tag.id"),primary_key = True)
)


class Article(Base):
    __tablename__ = 'article'
    id = Column(Integer,primary_key=True,autoincrement=True)
    title = Column(String(50),nullable=False)
    #tags = relationship("tag",backref="articles",secondary = article_tag )

    def __repr__(self):
        return "<Article(title:%s)>" % self.title

class Tag(Base):
    __tablename__ = 'tag'
    id = Column(Integer,primary_key=True,autoincrement=True)
    name= Column(String(50),nullable=False)
    articles = relationship("Article",backref="tags",secondary = article_tag)

    def __repr__(self):
        return "<Tag(title:%s)>" % self.title

#添加数据
article1 = Article(title="title1")
article2 = Article(title="title2")

tag1 = Tag(name="tag1")
tag2 = Tag(name="tag2")

article1.tags.append(tag1)
article1.tags.append(tag2)

article2.tags.append(tag1)
article2.tags.append(tag2)

session.add_all(article1,article2)
session.commit()

#查询数据
article = session.query(Article).first()
print(article.tags)
```
# ORM层面删除数据注意事项
尽管MySQL默认约束类型为RESTRIC，但若不设置从表中关联列的nullable=False，那么通过sqlalchemy的ORM删除数据不会报错！！
```python
class User(Base):
    __tablename__ = 'user'
    id = Column(Integer,primary_key=True,autoincrement=True)
    username= Column(String(50),nullable=False)
    articles = relationship("Article",backref="tags")

    def __repr__(self):
        return "<Tag(title:%s)>" % self.title

class Article(Base):
    __tablename__ = 'article'
    id = Column(Integer,primary_key=True,autoincrement=True)
    title= Column(String(50),nullable=False)
    uid = Column(Integer,ForeignKey("User.id"))
    author = relationship("User",backref = "articles")

    def __repr__(self):
        return "<Article(title:%s)>" % self.title

Base.metadata.drop_all()
Base.metadata.create_all()
#添加一条数据
user = User(username='zhiliao')
article = Article(title='hello world')
article.author = user
session.add(article)
session.commit()
#尽管MySQL默认约束类型为RESTRIC，但用sqlalchemy的ORM删除数据不会报错！！实际步骤为：检测到有外键关联，但发现外键类型为RESTRICT，因此先通过外键找到关联的数据，将关联的uid设置为null，之后再删除user。
user = session.query(User).first()
session.delete(user)
session.commit()
#要防止误删时RESTRICT没用，应该在创建外键时设置nullable=False。
class Article(Base):
    __tablename__ = 'article'
    id = Column(Integer,primary_key=True,autoincrement=True)
    title= Column(String(50),nullable=False)
    uid = Column(Integer,ForeignKey("User.id"),nullable=False)
    author = relationship("User",backref = "articles")

```
# relationship方法中的cascade参数详解(1-2)
## ORM层面的CASCADE
在SALAlchemy中，只要将一条数据添加到session中，和他相关联的数据都可以一起存入。这其实是通过relationship对象创建时传入关键字参数cascade实现的，另外，也可在backref中传入casecade关键字：
### `save-update`:
默认选项，自动把其他关联数据添加到数据库中就是这条实现的。若不想同步添加，令`cascade=''`
### `delete`：
表示当删除某一个模型中的数据的时候，是否也删掉使用`relationship`和他关联的数据。
### `delete-orphan`：
表示当对一个ORM对象解除了父表中的关联对象的时候，自己便会被删除掉(即子表中的某条数据没有了关联数据，则该条数据会被自动删除掉)。当然如果父表中的数据被删除，自己也会被删除。这个选项只能用在一对多上，不能用在多对多以及多对一上。并且还需要在子模型中的`relationship`中，增加一个`single_parent=True`的参数。
### `merge`：
默认选项。当在使用`session.merge`，合并一个对象的时候，会将使用了`relationship`相关联的对象也进行`merge`操作。如user表中已有一条数据id=1，而脚本中另外生成了一条id=1的数据，那么就可用merge覆盖掉原数据及相关联数据。实际项目使用较少。
### `expunge`：
移除操作的时候，会将相关联的对象也进行移除。这个操作与session.delete(obj)不同，它只是从`session`中移除，并等到`session.commit()`时不会真正的从数据库中删除。实际项目使用较少。
### `all`：
是对`save-update, merge, refresh-expire(极少用), expunge, delete`几种的缩写。
************
## save-update及delete例子

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
### delete-orphan
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
### merge合并操作及expunge移除操作因不常用，略。
# 三种排序方式详解
## 三种排序方法
### order_by
可以指定根据这个表中的某个字段进行排序，如果在前面加了一个‘-’，代表的是降序排序。正序排序为由小到大，倒序排序为由大到小。若针对时间排序，在倒序(从新往旧)排序时需调用desc()方法，或加负号。
```python
class User(Base):
    __tablename__ = 'user'
    id = Column(Integer,primary_key=True,autoincrement=True)
    username= Column(String(50),nullable=False)
    create_time = Column(DateTime,nullable=False,default=datetime.now)

articles = session.query(Article).order_by(Article.create_time.desc()).all()
#或
articles = session.query(Article).order_by("-create_time").all()

print(articles)
```
### 在模型定义的时候指定默认排序
有些时候，不想每次在查询的时候都指定排序的方式，可以在定义模型的时候就指定排序的方式。
有以下两种定义排序方式：

#### relationship的order_by参数
在指定relationship的时候，传递order_by参数来指定排序的字段。
```python
class User(Base):
    __tablename__ = 'user'
    id = Column(Integer,primary_key=True,autoincrement=True)
    username= Column(String(50),nullable=False)
    create_time = Column(DateTime,nullable=False,default=datetime.now)

class Article(Base):
    __tablename__ = 'article'
    id = Column(Integer,primary_key=True,autoincrement=True)
    title= Column(String(50),nullable=False)
    create_time = Column(DateTime,nullable=False,default=datetime.now)
    uid = Column(Integer,ForeignKey = "user.id"
    user = relationship("User",backref=backref("articles",order_by="-create_time")
```
#### 在模型定义中
添加以下代码：
```python
 __mapper_args__ = {
     "order_by": title
   }
```
或：
```
 __mapper_args__ = {
     "order_by": title.desc()
   }
```
即可让文章使用标题来进行排序。

### 正向排序和反向排序
默认情况是从小到大，从前到后排序的，如果想要反向排序，可以调用排序的字段的desc方法。
