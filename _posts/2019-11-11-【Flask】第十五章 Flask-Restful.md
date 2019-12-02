---
published: true
title: 【Flask】第十五章 Flask-Restful
category: flask
tags:
  - python
  - flask
layout: post
---
# Restful API规范介绍
`restful api`是用于在前端与后台进行通信的一套规范。使用这个规范可以让前后端开发变得更加轻松。以下将讨论这套规范的一些设计细节。
## 相关规定
### 协议
采用http或者https协议。
### 数据传输格式
数据之间传输的格式应该都使用json，而不使用xml。
### url链接
url链接中，不能有动词，只能有名词。并且对于一些名词，如果出现复数，那么应该在后面加s。
比如：获取文章列表，应该使用`/articles/`，而不应该使用/get_article/

### HTTP请求的方法
* GET：从服务器上获取资源。
* POST：在服务器上新创建一个资源。
* PUT：在服务器上更新资源。（客户端提供所有改变后的数据）
* PATCH：在服务器上更新资源。（客户端只提供需要改变的属性）
* DELETE：从服务器上删除资源。

示例：
* GET /users/：获取所有用户。
* POST /user/：新建一个用户。
* GET /user/id/：根据id获取一个用户。
* PUT /user/id/：更新某个id的用户的信息（需要提供用户的所有信息）。
* PATCH /user/id/：更新某个id的用户信息（只需要提供需要改变的信息）。
* DELETE /user/id/：删除一个用户。
### 状态码
|状态码|原生描述|描述|
|-----|--------|---|
200|OK|服务器成功响应客户端的请求。
400|INVALID REQUEST|用户发出的请求有错误，服务器没有进行新建或修改数据的操作
401|Unauthorized|用户没有权限访问这个请求
403|Forbidden|因为某些原因禁止访问这个请求
404|NOT FOUND|用户发送的请求的url不存在
406|NOT Acceptable|用户请求不被服务器接收（比如服务器期望客户端发送某个字段，但是没有发送）。
500|Internal server error|服务器内部错误，比如出现了bug

# Flask-Restful插件的基本使用
## Flask-Restful插件作用
如果想专门定义一个url用于返回json数据，则可用Flask-Restful插件定义一个类继承自`flask_restful.Resource`，之后绑定`url`映射，专门用于接收json数据；如果相渲染模板，则定义一个类继承`MethodView`，并绑定`url`映射。
## 使用方法
`pip install flask-restful`
如果使用`Flask-Restful`，那么定义视图函数的时候，就要继承自`flask_restful.Resource`类，然后再根据当前请求的`method`来定义相应的方法。比如期望客户端是使用`get`方法发送过来的请求，那么就定义一个`get`方法。类似于`MethodView`。
```py
from flask import Flask,render_template,url_for
from flask_restful import Api,Resource

app = Flask(__name__)
# 用Api来绑定app
api = Api(app)

class LoginView(Resource):
    def post(self):
        return {"username":"zhiliao"}

api.add_resource(LoginView,'/login/',endpoint='login')

with app.test_request_context():
    print(url_for('login'))

# api.add_resource(LoginView,'/login/')
# with app.test_request_context():
#     #反转时参数要小写！！
#     print(url_for('loginview'))


```

### 注意事项
* `endpoint`是用来给`url_for`反转`url`的时候指定的。如果不写`endpoint`，那么将会使用视图的名字的小写来作为`endpoint`
* `add_resource`的第二个参数是访问这个视图函数的`url`，这个`url`可以跟之前的`route`一样，可以传递参数。并且还有一点不同的是，这个方法可以传递多个`url`来指定这个视图函数。

## 参数传递与一对多url映射
```py
from flask import Flask,render_template,url_for
from flask_restful import Api,Resource

app = Flask(__name__)
# 用Api来绑定app
api = Api(app)

class LoginView(Resource):
    def post(self,username=None):
        return {"username":"zhiliao"}

api.add_resource(LoginView,'/login/<username>/','/regist/',endpoint='login')

with app.test_request_context():
    print(url_for('login',userame='zhiliao'))
    print(url_for('regist'))

```
## postman使用
postman用于模拟不同方法访问url，此处用于模拟post方法访问url。

# Flask-Restful参数验证
`Flask-Restful`插件提供了类似`WTForms`来验证提交的数据是否合法的包，叫做`reqparse`。以下是基本用法：
```py
parser = reqparse.RequestParser()
parser.add_argument('username',type=str,help='用户名验证错误！')
args = parser.parse_args()
```
## 解析数据——例子
```py
from flask import Flask,render_template,url_for
from flask_restful import Api,Resource,reqparse

app = Flask(__name__)
# 用Api来绑定app
api = Api(app)

class LoginView(Resource):
    def post(self):
        parser = reqparse.RequestParser()
        parser.add_argument('username',type=str,help='用户名验证错误！',default='abc',required=True)
        parser.add_argument('password',type=str,help='密码验证错误！')
        parser.add_argument('gender',type=str,choices=['male','female','secret'],help='性别类型不存在！')
        args = parser.parse_args()
        print(args) #是一个字典，包含解析的键与对应的值:{"username":"zhiliao","password":"zhiliao123"}
        return args

api.add_resource(LoginView,'/login/',endpoint='login')
```
再用postman，通过post方法给body栏传参`username=zhiliao,password=zhiliao123`进行post访问。
## 关键参数
* default：默认值，如果这个参数没有值，那么将使用这个参数指定的值。
* required：是否必须。默认为False，如果设置为True，那么这个参数就必须提交上来。
* type：这个参数的数据类型，如果指定，那么将使用指定的数据类型来强制转换提交上来的值，若转换不了，则会报错，返回help内容提示错误信息。
type有`[str,int,float]`等python内置的函数。
* choices：选项。提交上来的值只有满足这个选项中的值才符合验证通过，否则验证不通过。

* help：错误信息。如果验证失败后，将会使用这个参数指定的值作为错误信息。
* trim：是否要去掉前后的空格。
## type常用类型
### python内置数据类型
type可设置为`[str,int,float]`等python内置的函数
### `flask_restful.inputs`下的一些特定的数据类型来强制转换
#### url
会判断这个参数的值是否是一个url，如果不是，那么就会抛出异常。
```py
from flask_restful import inputs
parser.add_argument('home_page',type=inputs.url,help='url错误')
```
#### regex
正则表达式。
```py
from flask_restful import inputs
parser.add_argument('telephone',type=inputs.regex(r'1[3578]\d{9}'),help='手机号格式错误')
```
#### date
将这个字符串转换为datetime.date数据类型。如果转换不成功，则会抛出一个异常。
```py
from flask_restful import inputs
parser.add_argument('birthday',type=inputs.date,help='生日格式错误')
# inputs.date验证格式为 2019-10-13，只能接收一个同时包含年月日的参数
```
# Flask-Restful标准化返回参数(1)
对于一个视图函数，你可以指定好一些字段用于返回。以后可以使用ORM模型或者自定义的模型的时候，他会自动的获取模型中的相应的字段，生成json数据，然后再返回给客户端。这其中需要导入flask_restful.marshal_with装饰器。并且需要写一个字典，来指示需要返回的字段，以及该字段的数据类型。示例代码如下：
## 返回字典
```py
app = Flask(__name__)
api = Api(app)
from flask_restful import Api,Resource,fields,marshal_with
class ArticleView(Resource):
    resource_fields = {
        'username': fields.String,
        'age': fields.Integer,
        'school': fields.String
    }

    @marshal_with(resource_fields)
    def get(self):
        #因定义了resource_fields，则返回指定字典时会自动将resource_fields里的键值也一并返回给前端。若指定字典中某条只有键没有值，则默认值为None。
        return {"username":"zhiliao"}
api.add_resource(ArticleView,'/article/',endpoint='article')
```

## 返回模型对象
```py
from flask_restful import Api,Resource,fields,marshal_with
class Article(object):
    def __init__(self,username,age):
        self.username = username
        self.age = age

article = Article("zhiliao","18")

class ProfileView(Resource):
    resource_fields = {
        'username': fields.String,
        'age': fields.Integer,
        'school': fields.String
    }

    @marshal_with(resource_fields)
    def get(self):
        #返回对象时会自动解析该对象的绑定属性，并json成字典返回。
        return article
api.add_resource(ArticleView,'/article/',endpoint='article')
```
# Flask-Restful标准化返回参数(2)
## 重命名属性
很多时候你面向公众的字段名称是不同于内部的属性名。使用 attribute可以配置这种映射。比如现在想要返回user.school中的值，但是在返回给外面的时候，想以education返回回去，那么可以这样写：
```py
resource_fields = {
    'education': fields.String(attribute='school') #读取属性时使用school属性读取，但读取的属性名显示为education
}
```
## 默认值
在返回一些字段的时候，有时候可能没有值，那么这时候可以在指定fields的时候给定一个默认值，示例代码如下：
```py
resource_fields = {
    'age': fields.Integer(default=18)
}
```
## 复杂结构
有时候想要在返回的数据格式中，形成比较复杂的结构。那么可以使用一些特殊的字段来实现。比如要在一个字段中放置一个列表，那么可以使用fields.List，比如在一个字段下面又是一个字典，那么可以使用fields.Nested。以下将讲解下复杂结构的用法：
```py
class ProfileView(Resource):
    resource_fields = {
        'username': fields.String,
        'age': fields.Integer,
        'school': fields.String,
        'tags': fields.List(fields.String), #一个用户有多个标签
        'more': fields.Nested({
            'signature': fields.String,
            'ed_id':fields.Integer
        })
    }
```

# Flask-Restful细节强化
对于一个视图函数，你可以指定好一些字段用于返回。以后可以使用ORM模型或者自定义的模型的时候，他会自动的获取模型中的相应的字段，生成json数据，然后再返回给客户端。这其中需要导入flask_restful.marshal_with装饰器。并且需要写一个字典，来指示需要返回的字段，以及该字段的数据类型。示例代码如下：
```python
class ProfileView(Resource):
    resource_fields = {
        'username': fields.String,
        'age': fields.Integer,
        'school': fields.String
    }

    @marshal_with(resource_fields)
    def get(self,user_id):
        user = User.query.get(user_id)
        return user
```
在get方法中，返回user的时候，flask_restful会自动的读取user模型上的username以及age还有school属性。组装成一个json格式的字符串返回给客户端。


## Flask-Restful结合蓝图使用
1. 在蓝图中，如果使用`flask-restful`，那么在创建`Api`对象的时候，就不要再使用`app`了，而是使用蓝图。
2. 如果在`flask-restful`的视图中想要返回`html`代码，或者是模版，那么就应该使用`api.representation`这个装饰器来定义一个函数，在这个函数中，应该对`html`代码进行一个封装，再返回。示例代码如下：

```py
from flask import Blueprint,render_template,make_response
from flask_restful import Api,Resource,fields,marshal_with
from models import Article
import json
from flask import Response

article_bp = Blueprint('article',__name__,url_prefix='/article')
api = Api(article_bp)

# 这个代码是经过修改后的，能支持html和json
@api.representation('text/html')
def output_html(data,code,headers):
    if isinstance(data,str):
        # 在representation装饰的函数中，必须返回一个Response对象
        resp = make_response(data)
        return resp
    else:
        return Response(json.dumps(data),mimetype='application/json')

class ArticleView(Resource):
    resource_fields = {
        'aritlce_title':fields.String(attribute='title'),
        'content':fields.String,
        'author': fields.Nested({
            'username': fields.String,
            'email': fields.String
        }),
        'tags': fields.List(fields.Nested({
            'id': fields.Integer,
            'name': fields.String
        })),
        'read_count': fields.Integer(default=80)
    }

    @marshal_with(resource_fields)
    def get(self,article_id):
        article = Article.query.get(article_id)
        return article
# /article/article/1/
api.add_resource(ArticleView,'/<article_id>/',endpoint='article')

class ListView(Resource):
    def get(self):
        return render_template('index.html')
api.add_resource(ListView,'/list/',endpoint='list')
```
