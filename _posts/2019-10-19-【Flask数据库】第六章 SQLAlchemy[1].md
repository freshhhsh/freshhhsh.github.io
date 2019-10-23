---
published: true
title: 【Flask】第六章 SQLAlchemy[1]
category: flask_MySQL
tags:
  - python
  - flask
  - MySQL
layout: post
---
# sqlalchemy常用数据类型：
## sqlalchemy常用数据类型：

* Integer：整形(等同于INT)
*******

* Float：浮点类型（仅最多可存储32位数据，超出则丢失）
*******
* Double：双精度浮点类型（仅最多可存储64位数据，超出则丢失）
********
* Boolean: 传递True/False
*******
* DECIMAL: 定点数类型(无位数限制，可避免数据精度丢失问题)
使用需传入两个参数，第一个为总最大位数，第二个为小数点后最大位数
***********
* enum: 枚举类型
只枚举传入的变量名，如Article类下设置`tag = Column(Enum("python","flask","django"))`，则之后tag的值只能为python/flask/django。在python3中，包含enum包，通过继承enum.Enum类，可自定义枚举用类Myenum，之后只用传入`tag = Column(Enum(Myenum))`即可。
*********
* Date: 传递 datetime.date()进去
`from sqlalchemy import Date`
create_time = Column(Date)
date(年，月，日)
***********
* DateTime: 传递datetime.datetime()进去
`from sqlalchemy import Datetime`
create_time = Column(Datetime)
datetime(年，月，日[，时，分，秒，毫秒)]
*************
* Time: 传递datetime.time()进去
`from sqlalchemy import Time`
可以存储时、分、秒，time(hour=1,minute=1,second=1)
******
* String: 字符类型，使用时需要指定长度，区别于Text类型
`import...`
`title = Column(String(50))`
******
* Text: 文本类型(最多存6w多个字符)
`import...`
`title = Column(Text)`
*******
* LONGTEXT: 长文本类型(**仅MySQL拥有长文本类型**)
`from sqlalchemy.dialects.mysql import LONGTEXT`
*******
注：以上数据类型均需要在使用时从sqlalchemy库中导入。
*****
## 代码实现
===

* `Base.metadata.drop_all()`表示从数据库中删除目前绑定在Base类下的所有子类表。若数据库中本来就有其他子类表，而在此脚本中未绑定在Base下，则不会删除。再`Base.metadata.create_all()`则会把所有绑定在Base下的子类表重新在数据库中创建出来

```

#encoding: utf-8

from sqlalchemy import create_engine,Column,Integer,Float,Boolean,DECIMAL,Enum,Date,DateTime,Time,String,Text

from sqlalchemy.dialects.mysql import LONGTEXT

from sqlalchemy.ext.declarative import declarative_base

from sqlalchemy.orm import sessionmaker

# 在Python3中才有这个enum模块，在python2中没有

import enum

HOSTNAME = '127.0.0.1'

PORT = '3306'

DATABASE = 'first_sqlalchemy'

USERNAME = 'root'

PASSWORD = 'root'

# dialect+driver://username:password@host:port/database

DB_URI = "mysql+pymysql://{username}:{password}@{host}:{port}/{db}?charset=utf8mb4".format(username=USERNAME,password=PASSWORD,host=HOSTNAME,port=PORT,db=DATABASE)

engine = create_engine(DB_URI)

Base = declarative_base(engine)

session = sessionmaker(engine)()

class TagEnum(enum.Enum):

    python = "python"

    flask = "flask"

    django = "django"

class Article(Base):

    __tablename__ = 'article'

    id = Column(Integer,primary_key=True,autoincrement=True)

    # price = Column(Float)

    # is_delete = Column(Boolean)

    price = Column(DECIMAL(10,4)) #共存储10位，小数点后最多存储4位

    # 100000.0001

    # tag = Column(Enum(TagEnum))

    # create_time = Column(Date)

    # create_time = Column(DateTime)

    # create_time = Column(Time)

    # title = Column(String(50))

    # content = Column(Text)

    # content = Column(LONGTEXT)

# alembic

# flask-migrate

Base.metadata.drop_all()

Base.metadata.create_all()

from datetime import date

from datetime import datetime

from datetime import time

article = Article(price=100000.99999)

session.add(article)

session.commit()

```
# Column常用参数：

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
# query函数可查询的数据
## query可用参数：
### 模型对象。指定查找这个模型中的所有对象
```
articles = session.query(Article).all()
for article in articles:
    print(article)#重新实现__str__方法
```
### 模型中的属性。可以指定只查找某个模型的其中几个属性。
```
articles = session.query(Article.title,Article.price).all()#返回列表，其内元素为二元元组
print(articles)#因articles是列表，重新实现__repr__方法
```
### 聚合函数
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
# filter方法常用过滤条件
## 过滤条件
过滤是数据提取的重要功能，以下为filter方法实现的过滤:
******
### equals：等于
`session.query.filter(User.name == 'ed')`
*****
### not equals：不等于
`session.query.filter(User.name != 'ed')`
*****
### like：为Column的一个方法，表模糊匹配，like内传入匹配字段。
`session.query.filter(User.name.like('%ed%'))`
注：'%'为sql的通配符，匹配一个或多个字符。
        '_'仅替代一个字符。
        [abc]匹配abc中任意一个。
        [^abc]/[!abc]匹配非abc的任意字符。

注：ilike与like用法相同，但ilike不区分大小写
*****
### in_：判度胺是否存在
`session.query.filter(User.name.in_(['xushenghai','zhiliao']))`
*****
### not in:判断是否不存在
`session.query.filter(~User.name.in_(['xushenghai','zhiliao']))` #波浪表取反
或：
`session.query.filter(User.name.notin_(['xushenghai','zhiliao']))`
***
### is null:为空
先给数据库的article表添加一个空列content：
`alter table article add column content text`
再在脚本中加入content的Column对象
筛选是否为空:
`articles = session.query(Article).filter(Article.content == None).all()`
****
### is not null：不为空
`articles = session.query(Article).filter(Article.content != None).all()`
****
### and：并，可用逗号表示并,也可通过导入and_
`from sqlalchemy import and_`
`articles = session.query(Article).filter(and_(Article.content == 'abc', Article.title == 'abc')).all()`
或不用and_：
`articles = session.query(Article).filter(Article.content == 'abc', Article.title == 'abc').all()`
****
### or：或
`from sqlalchemy import or_`
`articles = session.query(Article).filter(or_(Article.content == 'abc', Article.title == 'abc')).all()`
****
## 备注
要想直接看orm底层转换的sql的命令语句，可以在filter方法后面不再调用.all()方法，直接打印结果即可。
```
articles = session.query(Article).filter(or_(Article.content == 'abc', Article.title == 'abc'))
print(articles)
```
# 外键及其四种约束
## 外键
通过ForeignKey类来实现，并且可以指定表的外键约束。
******
以下为user表为父表，article表为子表，通过user.id连接。
```python
class Article(Base):
    __tablename__ = 'article'
    id = Column(Integer,primary_key=True,autoincrement=True)
    title = Column(String(50),nullable=False)
    content = Column(Text,nullable=False)
    uid = Column(Integer,ForeignKey('user.id')) #此处数据类型应与父表的id的数据类型相同

    def __repr__(self):
        return "<Article(title:%s)>" % self.title

class User(Base):
    __tablename__ = 'user'
    id = Column(Integer,primary_key=True,autoincrement=True)
    username = Column(String(50),nullable=False)
```
****
### 外键约束种类：
#### RESTRICT：父表数据被删除，会阻止删除。默认就是这一项。
#### NO ACTION：在MySQL中，同RESTRICT。
#### CASCADE：级联删除。即父表数据被删除，子表也被删除。
#### SET NULL：父表数据被删除，子表数据会设置为NULL。
*****
### 选定外键约束的种类，通过设定关键字ondelete的值实现：
`uid = Column(Integer,ForeignKey('user.id',ondelete="RESTRICT"))`
则在mysql中输入`delete from user where id = 1;`会报错
# ORM层外键访问方式和一对多
## 一对多外键关系
通过sqlalchemy.orm下的relationship实现
如一个作者可以有多篇文章，此为一对多关系。以下为通过文章找作者和通过作者找文章的例子：
## 代码实现
```python
from sqlalchemy.orm import relationship
class User(Base):
    ...
    username = Column(...)
    articles = relationship("article")

class Article(Base):
    ...
    author = relationship("User")
#用relationship，通过article表数据取user表对应id的数据
article = session.query(Article).first()
print(article.author.username)
#用relationship，通过user表数据取article表对应id的数据
user = session.query(User).first()
print(user.articles)
```
反向引用可使relationship成双向关系，使用反向引用则不需要在每个类后面设定relationship。
```python
from sqlalchemy.orm import relationship
class User(Base):
    #...
    username = Column(...)

class Article(Base):
    #...
    author = relationship("User",beckref = "articles")
```
