---
published: true
title: 【Flask】第十三章 Flask上下文
category: flask
tags:
  - python
  - flask
layout: post
---
# Local线程隔离对象
## Local线程隔离对象
注：flask = werkzeug + salalchemy + jinja
* `werkzeug.local`中的`Local`类的对象，即使是同一个对象，在不同线程中其属性都是相互隔离的。
* `Thread Local`对象：只要满足其同一个对象的属性在多线程中是相互隔离的，则它就叫`Thread Local`对象。

flask中的request及session、g就是Thread Local的对象，具体为`werkzeug.local.Local`对象。由此实现不同线程之间相互隔离，以处理不同用户同时访问的问题。

# app上下文(应用上下文)和request上下文(请求上下文)详解
应用上下文和请求上下文都是存放到`LocalStack`的两个不同的对象栈中。和应用app相关的操作就必须要用到应用上下文。比如通过`current_app`获取当前`app`。和请求相关的操作就必须用到请求上下文，比如使用`url_for`反转函数。
## 为什么需要做将上下文推入栈中这件事
* 应用上下文：Flask底层是基于werkzeug，werkzeug是可以包含多个app的，所以这时候用一个栈来保存。如果你在使用app1，那么app1应该是要在栈的顶部，如果用完了app1，那么app1应该从栈中删除。方便其他代码使用下面的app。
* 如果在写测试代码，或者离线脚本的时候，我们有时候可能需要创建多个请求上下文，这时候就需要存放到一个栈中了。使用哪个请求上下文的时候，就把对应的请求上下文放到栈的顶部，用完了就要把这个请求上下文从栈中移除掉。


## app上下文
### 情况演示
`from flask import current_app`

在视图函数外：
`print(current_app.name)`会报错

在视图函数内：
`print(current_app.name)`会打印出主文件名

当访问视图函数时，flask会自动将AppContext压入LocalStack的对象中，此对象为一个装AppContext的栈，由此在该栈中就有了AppContext，因此current_app在访问栈后才能找出AppContext并打印出来。
### 手动将AppContext推入LocalStack对象栈中
```py
app_context = app.app_context()
app_context.push()
print(current_app.name)
```
或：
```py
with app.app_context():
    print(current_app.name)
```
## request上下文
### 情况演示
`from flask import url_for`

在视图函数内:
`print(url_for('my_list'))`

* 正常反转，在视图函数中不用担心上下文问题。执行视图函数肯定是通过访问url方式实现，这种情况下flask底层会自动将请求上下文和应用上下文推入到相应的栈中。

在视图函数外且在定义相应视图函数的代码之后：
`print(url_for('my_list'))`
* 报错，请求上下文栈中不存在请求上下文。

### 手动将RequestContext推入LocalStack对象栈中
手动推入请求上下文时，会首先判断有没有应用上下文，如果没有则推入应用上下文，之后再推入请求上下文。
```py
with app.test_request_context():
    print(url_for('my_list'))
```
# 线程隔离的g对象使用详解
g对象为单例对象，是global的缩写，一般用于保存flask以外的自定义全局数据。此外，他和request一样也是线程隔离的。

因g为flask的全局单例对象，因此在`context_demo.py`中绑定的g.username属性在其他脚本中也可以使用。

context_demo.py
```py
from flask import g
from utils import log_a,log_b,log_c
...
@app.route('/')
def index():
    username = request.args.get('username')
    log_a()
    log_b()
    log_c()
    return '首页'
```

utils.py
```py
from flask import g
#因g为flask的全局单例对象，因此在`context_demo.py`中绑定的g.username属性在其他脚本中也可以使用。
def log_a():
    print('this is %s'%g.username)

def log_b():
    print('this is %s'%g.username)

def log_c():
    print('this is %s'%g.username)

```
# before_request钩子函数详解
## before_first_request
服务器开启后处理第一次请求之前执行。例如以下代码：
```py
@app.before_first_request
def first_request():
    print('first time request')
```
## before_request
在每次请求之前执行。通常可以用这个装饰器来给视图函数增加一些变量。例如以下代码：
```py
@app.before_request
def before_request():
    # print('每次访问均执行钩子函数')
    user_id = session.get('user_id')
    if user_id:
        g.user = 'zhiliao'

@app.route('/')
def index():
    if hasattr(g,'user'):
        return '欢迎！%s'%g.user
    else:
        return '欢迎'
```
# context_processor钩子函数详解
## template_filter
在使用Jinja2模板的时候自定义过滤器。比如可以增加一个upper的过滤器

主文件中:
```py
@app.template_filter
    def upper_filter(s):
        return s.upper()
```
## context_processor
上下文处理器。返回的字典中的键可以在模板上下文中使用。

此处理器必须返回字典，而返回的字典内容会自动在`render_template`时传入模板中，由此省去每次`render_template`时手动设定传参的步骤，让代码简洁可维护。

* 必须返回字典!!!若字典没有键值对可返回空

```py
@app.route('/')
def index():
    return render_template("index") #会自动将current_user="zhiliao"传进去


@app.context_processor
def context_processor():
  if hasattr(g,'user'):
    return {"current_user":g.user}
  else:
    return {}
```

index.html:
```html
<body>
    {{ current_user }}
</body>
```
# errorhandle钩子函数详解及abort函数
errorhandler接收状态码，装饰的函数传入`error`，则可以自定义返回这种状态码的响应处理方法。返回处理方法+逗号+状态码，才可让客户端获取到正确的状态码，否则会获取正常状态码200。
## errorhandle钩子函数
```py
@app.errorhandler(404)
def page_not_found(error):
    return 'This page does not exist',404
    #或render_template(),状态码
@app.errorhandler(500)
def server_error(error):
    return render_template('500error.html'),500
```
## abort函数
在视图函数中调用，传入状态码后会主动返回报错信息。
```py
from flask import abort

@app.route('/')
def index():
    abort(404)
    return render_template('index.html')
```
