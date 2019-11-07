---
published: true
title: 【Flask】第十一章 cookie与session
category: flask_MySQL
tags:
  - python
  - flask
layout: post
---
# cookie的基本概念
## cookie
在网站中，http请求是无状态的。也就是说即使第一次和服务器连接后并且登录成功后，第二次请求服务器依然不能知道当前请求是哪个用户。cookie的出现就是为了解决这个问题，第一次登录后服务器返回一些数据（cookie）给浏览器，然后浏览器保存在本地，当该用户发送第二次请求的时候，就会自动的把上次请求存储的cookie数据自动的携带给服务器，服务器通过浏览器携带的数据就能判断当前用户是哪个了。cookie存储的数据量有限，不同的浏览器有不同的存储大小，但一般不超过4KB。因此使用cookie只能存储一些小量的数据。
## 注意项
* cookie有有效期，过期则浏览器会自动清除。
* cookie有域名的概念，只有访问同域名才会返回相应的cookie。

# Flask设置和删除cookie
## flask操作cookie
### 设置cookie
* 应该在Response对象上调用set_cookie方法，之后返回这个Response对象

```py
from flask import Flask,request,Response
app = Flask(__name__)
@app.route('/')
def hello_world():
    resp = Response("知了课堂")
    '''
    set_cookie(self, key(用于识别用户的参数名), value=''(用于识别用户的参数值),
    max_age=None(过期时间，秒), expires=None,
    path='/'(设置成斜杠代表当前域名下所有url都可使用改cookie),
    domain=None,
    secure=False(http协议时为False,https协议时为True),
    httponly=False(False表可以在前端使用JavaScript读取cookie及操作))
    '''
    resp.set_cookie('username','zhiliao')   
    return resp
```
#### 查看cookie方法
* 访问某服务器主页，开发者模式中在Network栏，再刷新页面，可看到访问，在Response Headers中可看到Set-Cookie: ...
* chrome点击url左边的信息icon，找到相应域名再展开查看。
* chrome中设置>高级设置>内容设置>所有cookie  (当cookie改变时，此种方法需要关闭设置再开启才会刷新cookie记录)
### 删除cookie
* 通过delete_cookie(self,key)方法，指定标识用户的变量名来删除。
```py
from flask import Flask,request,Response
app = Flask(__name__)
@app.route('/')
def hello_world():
    resp = Response("知了课堂")
    resp.set_cookie('username','zhiliao')   
    return resp
@app.route('/del_cookie/')
def del_cookie():
    resp = Response("删除cookie")
    #delete_cookie(self,key)
    resp.delete_cookie('username')
    return resp
```
# Flask设置cookie过期时间
## Flask设置cookie过期时间
若不设置过期时间，则默认过期时间为：浏览会话结束时就过期，即浏览器关闭后过期
### max_age:在IE8以下不支持
max_age=None(过期时间，秒)
```py
from flask import Flask,request,Response
from datetime import datetime
app = Flask(__name__)
@app.route('/')
def hello_world():
    resp = Response("知了课堂")
    '''
    set_cookie(self, key(用于识别用户的参数名), value=''(用于识别用户的参数值),
    max_age=None(过期时间，秒), expires=None(接收datetime()),
    path='/'(设置成斜杠代表当前域名下所有url都可使用改cookie),
    domain=None,
    secure=False(http协议时为False,https协议时为True),
    httponly=False(False表可以在前端使用JavaScript读取cookie及操作))
    '''
    resp.set_cookie('username','zhiliao', max_age=60*60*24*7)
    return resp
```
### expires:新版本http协议中，expires参数被废弃
expires=None，接收datetime(),如datetime(year=2019,month=12,day=12,hour=0,minute=0,second=0)
datetime.now() + timedelta(days=30,hours=16)

* 注意：expires为格林尼治时间，所以在中国，浏览器会自动将expires设置的时间延后八小时！！！因此在设置目标北京时间时，需预先减八个小时

方法一：

```py
expires = datetime(year=2019,month=12,day=11,hour=16,minute=0,second=0)
resp.set_cookie('username','zhiliao', expires = expires)   
```
则该条cookie显示过期时间为2019年12月12日上午12点（即0点）0分0秒

方法二：
```py
from datetime import datetime,timedelta
expires = datetime.now() + timedelta(days=30)
resp.set_cookie('username','zhiliao', expires = expires)   
```
# 设置cookie的有效域名
## 应用蓝图实现子域名，进而设置cookie的有效域名
cookie_demo.py
```py
from flask import Flask,request,Response
from datetime import datetime
from csmviews import bp
app = Flask(__name__)
app.register_blueprint(bp)
app.config['SERVER_NAME'] = 'hy.com:5000'

@app.route('/')
def hello_world():
    resp = Response("知了课堂")
    '''
    set_cookie(self, key(用于识别用户的参数名), value=''(用于识别用户的参数值),
    max_age=None(过期时间，秒), expires=None(接收datetime()),
    path='/'(设置成斜杠代表当前域名下所有url都可使用改cookie),
    domain=None,
    secure=False(http协议时为False,https协议时为True),
    httponly=False(False表可以在前端使用JavaScript读取cookie及操作))
    '''
    resp.set_cookie('username','zhiliao',domain='.hy.com') #domain参数为设置域名，前面加点'.'代表可在子域名中使用该cookie
    return resp
```

cmsviews.py
```py
from flask import Blueprint,request

bp = Blueprint('cms',__name__,subdomain='cms')

@bp.route('/')
def index():
    username = request.cookies.get('username')
    return username or '未获取到cookie'
```
## 要点
* 设置cookie的有效域名：cookie默认只能在主域名下使用，若需要在子域名中也使用，则需要在设置cookie时传入域名domain，且domain的值为'.'+'主域名',
如`set_cookie(key,value='',domain='.csdn.net')``

# session的基本概念
## 基本概念
session和cookie的作用有点类似，都是为了存储用户相关的信息。不同的是，cookie是存储在本地浏览器，session是一个思路、一个概念、一个服务器存储授权信息的解决方案，不同的服务器，不同的框架，不同的语言有不同的实现。虽然实现不一样，但是他们的目的都是服务器为了方便存储数据的。session的出现，是为了解决cookie存储数据不安全的问题的。
## session和cookie的结合使用介绍
cookie和session结合使用：web开发发展至今，cookie和session的使用已经出现了一些非常成熟的方案。在如今的市场或者企业里，一般有两种存储方式：

### session存储在服务器端
服务器端可以采用mysql、redis、memcached等来存储session信息。原理是：客户端发送验证信息到session中，然后随机生成一个唯一的session_id，再把这个session_id存储cookie中返回给浏览器，浏览器在以后请求服务器时，就把这个session_id自动的发送给服务器，服务器再从cookie中提取session_id，然后从服务器的session容器中找到这个用户的相关信息。

### session存储在客户端
原理是客户端发送验证信息（如用户名和密码）过来，服务器把相关的验证信息进行加密，然后再把加密后的信息存储到cookie返回给浏览器。浏览器再次访问时，自动发送cookie给服务器，服务器从cookie中提取session信息，进而识别用户。
# Flask操作session
flask默认使用session存储在客户端的方式操作session
## 设置、获取、删除、清空session
新建项目session_demo

session_demo.py
```py
from flask import Flask,session
import os

app = Flask(__name__)
#设置session的加密键
app.config['SECRET_KEY'] = os.urandom(24)

#设置session
@app.route('/')
def index():
    #设置session的键值后，flask会将他会自动加密后加入到cookie中
    session['username'] == 'zhiliao' #session为一个字典
    return 'Hello World'

#获取session
@app.route('/get_session/')
def get_session():
    username = session.get('username') #使用get方法更安全，不会抛出异常，未获取到则返回None
    return username or '未获取到username'

#删除session
@app.route('/delete_session/')
def delete_session():
    session.pop('username')
    # del session['username']
    return '删除成功'

#清空session
@app.route('/clear_session/')
def clear_session():
    session.clear()
    return '全部删除成功'

#设置session有效期
```
## 设置session有效期
* 方法一：`session.permanent = True`默认保存31天
```py
@app.route('/')
def index():
    session['username'] = 'zhiliao'
    session.permanent = True #变为True后默认可保存31天
    return 'Hello World!'
```
* 方法二：
```py
from datetime import timedelta
app.config['PERMANENT_SESSION_LIFETIME'] = timedelta(hours=2)
@app.route('/')
def index():
    session['username'] = 'zhiliao'
    session.permanent = True #变为True后默认可保存31天
    return 'Hello World!'
```
