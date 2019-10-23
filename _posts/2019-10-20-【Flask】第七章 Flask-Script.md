---
published: true
title: 【Flask】第七章 Flask-Script详细讲解
category: flask_MySQL
tags:
  - python
  - flask
  - MySQL
layout: post
---
# Flask-Script详细讲解
## Flask-Script介绍
`Flask-Script`的作用是可以通过命令行的形式来操作`Flask`。例如通过命令跑一个开发版本的服务器、设置数据库，定时任务等。要使用`Flask-Script`，可以进入`虚拟环境`后通过`pip install flask-script`安装最新版本。首先看一个最简单的例子：
```python
# manage.py

from flask_script import Manager
from your_app import app

manager = Manager(app)

@manager.command  #将greet转化为一个命令提示符的命令
def greet():
    print 'hello'


if __name__ == '__main__':
    manager.run()
```
命令提示符中：
```shell
workon flask-env
python manage.py greet
```
## 定义命令的三种方法
### 使用@command装饰器
这种方法上面已经介绍过。就不过多讲解了。

### 使用option装饰器传参
如果想要在使用命令的时候还传递参数进去，那么使用@option装饰器更加的方便。
```py
@manager.option('-n','--name',dest='name') #-n为指定--name的简化,dest为指定参数
@manager.option('-a','--age',dest='age') #-a为指定--age的简化,dest为指定参数
def hello(name):
    print('hello %s, I\'m %s years old.'%(name,age))
```
这样，调用hello命令：
```shell
python manage.py -n xt -a 18
```
或者
```shell
python manage.py --name xt -age 18
```
就可以输出：
`hello xt, I'm 18 years old`

##实际用法
* 用option方法装饰自定义`add_user(name,email)函数`，使得可以使用命令行添加管理员用户，比较安全与方便。
```python
@manager.option("-u","--username",dest="username")
@manager.option("-e","--email",dest="email")
def add_user(username,email):
    user = BackendUser(username = username,email = email)
    db.session.add(user)
    db.session.commit()
```
* 在项目中创建一个db_script.py，里面专门装自定义命令行命令。
db_script.py中：
```py
from flask_script import manager

db_manager = Manager()

@db_manager.command
def init():
    print("OK")

@db_manager.command
def revision():
    print("OK")

@db_manager.command
def upgrade():
    print("OK")
```
manager.py中：
```py
from db_script import db_manager
manager = Manager(app)
manager.add_command("db",db_manager)  #db为自定义命令
```
在终端上：
`python manager db init`
`python manager db upgrade`
