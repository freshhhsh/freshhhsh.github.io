---
published: true
title: 【Flask】第六章 SQLAlchemy[3]
category: flask_MySQL
tags:
  - python
  - flask
  - MySQL
layout: post
---
# limit、offset以及切片操作
## limit：可以限制每次查询的时候只查询几条数据。
```python
class Article(Base):
    __tablename__ = 'article'
    id = Column(Integer,primary_key=True,autoincrement=True)
    title = Column(String(50),nullable=False)
    create_time = Column(DateTime,default=datetime.now)

articles = session.query(Article).limit(10).all()
```
## offset：可以限制查找数据的时候过滤掉前面多少条。
```python
articles = session.query(Article).order_by(Article.id.desc()).offset(10).limit(10).all() #从第二十篇开始，往前找10篇
```
## slice切片：可以对Query对象使用切片操作，来获取想要的数据。      
1)`slice(self,start,stop)`start从0开始，含，stop不含。
```python
articles = session.query(Article).order_by(Article.id.desc()).slice(0,10).all()
```
2)**[  ]:常用方法**
```python
articles = session.query(Article).order_by(Article.id.desc())[0:10]
```
# 数据查询懒加载技术
## 懒加载
在一对多，或者多对多的时候，如果想要获取多的这一部分的数据的时候，往往能通过一个属性就可以全部获取了。比如有一个作者，想要或者这个作者的所有文章，那么可以通过user.articles就可以获取所有的。但有时候我们不想获取所有的数据，比如只想获取这个作者今天发表的文章，那么这时候我们可以给relationship传递一个lazy='dynamic'，以后通过user.articles获取到的就不是一个列表，而是一个AppendQuery对象了。这样就可以对这个对象再进行一层过滤和排序等操作。
******
## lazy可用选项
### 'select'：默认选项，如果不使用访问对象的某些属性，则不会提取那些属性，如user.articles，在不使用时就不会访问，一旦使用则提取该属性的全部内容并组装成一个列表。
### 'dynamic'：访问'user.articles'不是返回一个列表，而是`AppenderQuery`对象。
### `immediate`：立刻提取访问对象并提取它的相关属性。
## 代码实现
```python
class User(Base):
    __tablename__ = 'user'
    id = Column(Integer,primary_key = True,autoincrement = True)
    username = Column(String(50),nullable = False)

class Article(Base):
    __tablename__ = article
    id = Column(Integer,primary_key = True,autoincrement = True)
    title = Column(String(50),nullable = False)
    create_time = Column(DateTime,default = datetime.now)
    uid = Column(Integer,ForeignKey("user.id"))

    autho = relationship("User",backref=backref("articles",lazy = 'dynamic')

user  = session.query(User).first()
#因为使用了lazy='dynamic'，所以user为AppenderQuery对象，可继续追加数据进去。
article = Article(title="title 101")
user.articles.append(article)
session.commit()
```
# group_by和having子句
## group_by
根据某个字段进行分组。比如想要根据性别进行分组，来统计每个分组分别有多少人，那么可以使用以下代码来完成：
```
session.query(User.gender,func.count(User.id)).group_by(User.gender).all()
```
## having
having是对查找结果进一步过滤。比如只想要看未成年人的数量，那么可以首先对年龄进行分组统计人数，然后再对分组进行having过滤。示例代码如下：
```
result = session.query(User.age,func.count(User.id)).group_by(User.age).having(User.age >= 18).all()
```
# join实现复杂查询
## join方法：
join查询分为两种，一种是inner join，另一种是outer join。默认的是inner join，如果指定left join或者是right join则为outer join。
### 内连接：inner join
两张被内连接的表主键不匹配时，不匹配的数据均被抛弃。
### 外连接：left join
是以左表为基准，连接过程中右表不匹配项会被设置为null，左表数据保留；right join是以右表为基准，连接过程中左表不匹配项会被设置为null，右表数据保留。
#### 通过原生sql语句实现：
```python
select user.username,count(article.id) from user join article on user.id = article.uid group by user.id order by count(article.id) desc;
从join后的user与article中提取数据，显示不同user的article数量并以降序的方式排列出来。
```
#### 如果想要查询User及其对应的Article，则可以通过以下方式来实现(默认join为内连接)：
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
# subquery实现复杂查询
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
## query传统方式分两次查找
```python
user = session.query(User).filter(User.username = "A").first()
users = session.query(User).filter(User.city == user.city,User.age = user.age).all()
```
## sql原生传统方式子查找
```python
select user.id,user.username,user.city,user.age from user,(select user.city,user.age from user where user.username="A") as li_a where user.age = li_a.age and user.city=li_a.city
```
## query子查寻方式：
```python
stmt = session.query(User.city.lable("city"),User.age.lable("age")).filter(User.username=="A").subquery()
session.query(User).filter(User.city ==stmt.c.city,User.age==stmt.c.age).all()
```
调用`subquery()`方法后会返回子查询对象，用lable生成对应属性的别名方便以后调用(`age`和`city`属性)，用`stmt`变量接收后，可以调用`.c`方法，`c`代表`Column`列，`stmt.c.city`返回子查询对象的city值。
# `Flask-SQLAlchemy`插件的使用
## `Flask-SQLAlchemy`插件
功能为将创建引擎部分做成插件，供flask使用，以连接数据库。
* 用`db.Model`作为基类创建类表。
* `Column\Integer\String\relationship`不再需要导入，用`db.Column`形式就可以。
* 在定义模型的时候。可以不写`__tablename__`，则`Flask-SQLAlchemy`会自动将表名设为类名的小写。驼峰命名的类则会转为小写后用下划线连接，如`UserModel`转为`user_model`。明言胜于暗语，此条功能不用为好！！！
* 如果只是查找一个表上的数据，可以通过`表名.query`方法查询。如
`users = User.query.order_by(User.id.desc())all()`

## `Flask-SQLAlchemy`插件安装
`pip install flask-sqlalchemy`
```python
from flask import Flask
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)

HOSTNAME = '127.0.0.1'
PORT = '3306'
DATABASE = 'flask_sqlalchemy_demo'
USERNAME = 'root'
PASSWORD = 'root'
# dialect+driver://username:password@host:port/database
DB_URI = "mysql+pymysql://{username}:{password}@{host}:{port}/{db}?charset=utf8mb4".format(username=USERNAME,password=PASSWORD,host=HOSTNAME,port=PORT,db=DATABASE)

app.config['SQLALCHEMY_DATABASE_URL'] = DB_URI
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False

db = SQLAlchemy(app)

class User(db.Model):
    #flask-sqlalchemy插件会默认动将表名设为类名的小写，此行可以省略。
    #__tablename__ = 'user'
    #若类名为驼峰命名模式，则改成小写后用下划线连接：如UserModel表名默认为user_model。
    #一般不省略指定表名！！！

    id = db.Column(db.Integer,primary_key = True,autoincrement = True)
    username = db.Column(db.String(50),nullable = False)

    def __repr__(self):
        return 'User(username: %s)'%self.username

class Article(db.Model):
    id = db.Column(db.Integer,primary_key =True,autoincrement = True)
    title = db.Column(db.String(50),nullalble=False)
    uid = db.Column(db.Integer,db.ForeignKey("user.id"))

    author = db.relationship("User",backref="articles")

db.drop_all()
db.create_all()

#添加数据
user = User(username="user1")
article = Article(title="title1")
article.author = user

db.session.add(article)
db.session.commit()

#查询数据
#group_by
#order_by
#filter
#join
users = User.query.all()
article = Article.query.order_by(User.id.desc()).all()
print(users)

#删除、修改
user = User.query.filter(User.username == 'zhiliao').first()
user.username = 'zhiliao1'
db.session.commit()
db.session.delete(user)
db.session.commit()

@app.route('/')
def hello_world():
    return "Hello World!"

if __name__ == "__main__":
    app.run()
```
