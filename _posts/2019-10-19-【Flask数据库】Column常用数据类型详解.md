---
published: true
title: 【Flask数据库】Column常用数据类型详解
category: flask_MySQL
tags:
  - python
  - flask
  - MySQL
layout: post
---
# 一、sqlalchemy常用数据类型：

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
二、代码实现
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
