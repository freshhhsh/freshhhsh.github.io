---
published: true
title: 【Flask】第九章 Flask-Migrate项目结构重构
category: flask_MySQL
tags:
  - python
  - flask
  - MySQL
layout: post
---
# Flask-Migrate项目结构重构
## 项目结构重构
将项目表定义放入`models.py`中；数据库配置信息放入`config.py`中；主运行文件相关代码放入`zhiliao.py`中；为防止循环引用，设置中间模组`exts.py`。


config.py:
```py
DB_HOST = '127.0.0.1'
DB_PORT = '3306'
DB_NAME = 'flask_migrate_demo'
DB_USERNAME = 'root'
DB_PASSWORD = 'root'
DB_URI = "mysql+pymysql://{username}:{password}@{host}:{port}/{db}?charset=utf8mb4".format(username=DB_USERNAME,password=DB_PASSWORD,host=DB_HOST,port=DB_PORT,db=DB_NAME)
SQLALCHEMY_DATABASE = DB_URI
```




zhiliao.py:
```py
from flask import Flask
import config
from models import User
from exts import db

app = Flask(__name__)
app.config.from_object(config)
db.init_app(app)

@app.route('/')
def hello_world():
    return 'Hello World!'

@app.profile('/profile/')
def profile():
    pass


if __name__ == '__main__':
  app.run()
```

models.py:
```py
from exts import db
class User(db.Model):
    __tablename__ = 'user'
    id = db.Column(db.Integer,primary_key = True,autoincrement = True)
    username = db.Column(db.String(50),nullable=False)
```

exts.py:
```py
from flask_sqlalchemy import SQLAlchemy
db = SQLAlchemy()
```
---
# Flask-Migrate详细讲解
## Flask-Migrate介绍
在实际的开发环境中，经常会发生数据库修改的行为。一般我们修改数据库不会直接手动的去修改，而是去修改`ORM`对应的模型，然后再把模型映射到数据库中。这时候如果有一个工具能专门做这种事情，就显得非常有用了，而`flask-migrate`就是做这个事情的。`flask-migrate`是基于`Alembic`进行的一个封装，并集成到`Flask`中，而所有的迁移操作其实都是`Alembic`做的，他能跟踪模型的变化，并将变化映射到数据库中。
### 安装
虚拟环境中：`pip install flask-migrate`
### 绑定数据库及app
要让`Flask-Migrate`能够管理`app`中的数据库，需要使用`Migrate(app,db)`来绑定`app`和数据库。接上节项目代码。

manage.py:
```py
from flask_script import Manager
from zhiliao import app
from exts import db
from flask_migrate import Migrate,MigrateCommand
from models import User

manager = Manager(app)
Migrate(app,db)
manager.add_command("db",MigrateCommand)

if __name__ == '__main__':
  app.run()
```

### 命令提示符的migrate使用命令
接下来就是`cd`到项目目录，虚拟环境下:
* 将当前的app导入到环境变量中后，需要初始化一个迁移文件夹：
`python manage.py db init`  
* 然后再把当前的模型添加到迁移文件中：
`python manage.py db migrate`
* 最后再把迁移文件中对应的数据库操作，真正的映射到数据库中：
`python manage.py db upgrade`

### 其他migrate命令
`python manager.py db --help`
