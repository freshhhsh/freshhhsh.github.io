---
published: true
title: 【Flask数据库】alembic数据库迁移工具基本使用
category: flask_MySQL
tags:
  - python
  - flask
  - MySQL
layout: post
---
@[TOC](alembic数据库迁移工具基本使用)
# 一、alembic
## 1、alembic介绍
`alembic`是`sqlalchemy`的作者开发的。用来做`OMR`模型与数据库的迁移与映射。`alembic`使用方式跟`git`有点了类似，表现在两个方面，第一个，`alembic`的所有命令都是以`alembic`开头；第二，`alembic`的迁移文件也是通过版本进行控制的。首先，进入虚拟环境`workon flask-env`，通过`pip install alembic`进行安装。
### 1)作用流程
	ORM ——→ 迁移脚本 ——→ 数据库
## 2、用法
创建一个新的纯python项目，新建modes.py文件。
### 1）初始化alembic仓库
在终端中，先使用项目所用的虚拟环境`workon flask-env`，再`cd`到你的项目目录，执行`alembic init alembic`，即创建名叫`alembic`的仓库。
### 2）创建模型类
在项目中创建`models.py`模块，内容如下：
```python
#encoding: utf-8
from sqlalchemy import Column,String,Integer,create_engine
from sqlalchemy.ext.declarative import declarative_base

DB_HOST = '127.0.0.1'
DB_PORT = '3306'
DB_NAME = 'alembic_demo'
DB_USERNAME = 'root'
DB_PASSWORD = 'root'
DB_URI = "mysql+pymysql://{username}:{password}@{host}:{port}/{db}?charset=utf8mb4".format(username=DB_USERNAME,password=DB_PASSWORD,host=DB_HOST,port=DB_PORT,db=DB_NAME)

engine = create_engine(DB_URI)
Base = declarative_base(engine)

class User(Base):
	__tablename__ = 'user'
	id = Column(Integer,primary_key = True,autoincrement = True)
	username = Column(String(50),nullable = False)
```
### 3）修改配置文件：
* 再`alembic.ini`中设置数据库的连接，格式为：`sqlalchemy.url = driver://user:pass@localhost/dbname`，如：
	`sqlalchemy.url = mysql+pymysql://root:root@localhost/alembic_demo?charset=utf8`
* 为了使模型可以更新数据库，需要再`env.py`文件中设置`target_metadata`，默认为`None`，需设置为
```python
import sys#此条是为了将env.py的目录加入环境查询目录。
import models#为了使用models的Base。
target_metadata=models.Base.metadata
```
或者：
```python
import sys,os
sys.path.append(os.path.dirname(os.path.dirname(__filename__)))
import models
target_metadata=models.Base.metadata
```
### 4）自动生成迁移脚本
使用`alembic revision --autogenerate -m "message"`将当前模型中的状态生成迁移文件。
### 5）更新数据库
使用`alembic upgrade head`将刚刚生成的迁移文件，真正映射到数据库中。同理，如果要降级，那么使用`alembic downgrade head`。
### 6）修改代码后，重复4~5的步骤。
### 7）命令和参数解释
* init：创建一个alembic仓库。
* revision：创建一个新的版本文件。
* --autogenerate：自动将当前模型的修改，生成迁移脚本。
* -m：备注本次迁移做了哪些修改，用户可以自定义指定这个参数，方便回顾时给予提示。
* upgrade：将指定版本的迁移文件映射到数据库中，会执行版本文件中的upgrade函数。如果有多个迁移脚本没有被映射到数据库中，那么会执行多个迁移脚本。
* [head]：代表最新的迁移脚本的版本号。
* downgrade：会执行指定版本的迁移文件中的downgrade函数。
* heads：展示head指向的脚本文件版本号。
* history：列出所有的迁移版本及其信息。
* current：展示当前数据库中的版本号。
