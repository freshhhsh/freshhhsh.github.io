---
published: true
title: 【Flask】第十四章 flask信号机制
category: flask
tags:
  - python
  - flask
layout: post
---
# 信号及其使用场景详解
flask中的信号使用的是一个第三方插件，叫做blinker。通过pip list看一下，如果没有安装，通过以下命令即可安装blinker：
`pip install blinker`
## 自定义信号
自定义信号分为3步，第一是定义一个信号，第二是监听一个信号，第三是发送一个信号。以下将对这三步进行讲解：
### 定义信号
定义信号需要使用到`blinker`这个包的`Namespace`类来创建一个命名空间。比如定义一个在访问了某个视图函数的时候的信号。
```py
from blinker import Namespace

mysignal = Namespace()
visit_signal = mysignal.signal('visit-signal')
```

### 监听信号
监听信号使用`singal`对象的`connect`方法，在这个方法中需要传递一个函数，用来接收以后监听到这个信号该做的事情。
```py
def visit_func(sender,username):
    print(sender)
    print(username)

#使以后接收到信号后立刻执行传入的函数visit_func
mysignal.connect(visit_func)
```
### 发送信号
发送信号使用`singal`对象的`send`方法，这个方法可以传递一些其他参数过去。示例代码如下：
```py
mysignal.send(username='zhiliao')
```
## 自定义信号例子
用信号的方式记录用户登陆的信息，便于封装与结构化。
signal_demo.py:
```py
from flask import Flask,request
from signals import login_signal
app = Flask(__name__)
@app.route('/')
def index():
    return 'hello world!'

@app.route('/login/')
def login():
    #通过在url中后跟?username=XXX的方式，即查询字符串的方式
    username = request.args.get('username')
    if username:
        #获取成功则发送信号
        #方法一：手动传参
        #login_signal.send(username=username)
        #方法二：g对象自动传参
        g.username = username
        login_signal.send()
        return '登陆成功'
    else:
        return '请输入用户名'
```

signals.py:
```py
from blinker import Namespace
from datetime import datetime
from flask import request,g

namespace = Namespace()

login_signal = namespace.signal('login')

def login_log(sender):
    username = g.username
    now = datetime.now()
    ip = request.remote_addr
    log_line = "{username}*{now}*{ip}".format(username=username,now=now,ip=ip)
    #在pycharm中无法以管理员身份运行脚本，因此需手动用管理员模式打开cmd，虚拟环境中手动运行脚本
    with open("loging_log.txt",'a') as fp:
        fp.write(log_line+'/n')

login_signal.connect(login_log)
```

# Flask内置的信号
## flask内置常用的信号
### `flask.template_rendered`
模版渲染完毕后自动发送信号
```py
from flask import template_rendered
def log_template_renders(sender,template,context):
    print('sender:',sender) #发送者
    print('template:',template) #模板名
    print('context:',context) #其他参数

template_rendered.connect(log_template_renders,app)
```
### flask.got_request_exception
在请求过程中抛出异常时发送，异常本身会通过exception传递到订阅的函数。
```py
from flask import got_request_exception
def request_exception_log(sender,*args,**kwargs):
    print(sender)
    print(args)
    print(kwargs)
got_request_exception.connect(request_exception_log)
```
### 常用内置信号列表
1. `template_rendered`：模版渲染完成后的信号。
2. `before_render_template`：模版渲染之前的信号。
3. `request_started`：模版开始渲染。
4. `request_finished`：模版渲染完成。
5. `request_tearing_down`：request对象被销毁的信号。
6. `got_request_exception`：视图函数发生异常的信号。一般可以监听这个信号，来记录网站异常信息。
7. `appcontext_tearing_down`：app上下文被销毁的信号。
8. `appcontext_pushed`：app上下文被推入到栈上的信号。
9. `appcontext_popped`：app上下文被推出栈中的信号
10. `message_flashed`：调用了Flask的`flashed`方法的信号。
