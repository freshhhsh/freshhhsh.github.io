---
published: true
title: 【Flask数据库】`Flask-SQLAlchemy`插件的使用
category: flask_数据库
tags:
  - python
  - flask
  - MySQL
layout: post
---
@[TOC](文章目录)
###`Flask-SQLAlchemy`插件的功能为将创建引擎部分做成插件，供flask使用，以连接数据库。
###1、用`db.Model`作为基类创建类表。
###2、`Column\Integer\String\relationship`不再需要导入，用`db.Column`形式就可以。
###3、在定义模型的时候。可以不写`__tablename__`，则`Flask-SQLAlchemy`会自动将表名设为类名的小写。驼峰命名的类则会转为小写后用下划线连接，如`UserModel`转为`user_model`。明言胜于暗语，此条功能不用为好！！！
###4、如果只是查找一个表上的数据，可以通过`表名.query`方法查询。如
`users = User.query.order_by(User.id.desc())all()`

###安装：
`pip install flask-sqlalchemy`
```python-flask
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
