---
published: true
title: 【Flask】第十章 Flask_WTForms
category: flask_MySQL
tags:
  - python
  - flask
layout: post
---
# WTForms表单验证基本使用
`Flask-WTF`是简化了`WTForms`操作的一个第三方库。`WTForms`表单的两个主要功能是验证用户提交数据的合法性以及渲染模板。当然还包括一些其他的功能：`CSRF`保护，文件上传等。安装`Flask-WTF`默认也会安装`WTForms`，因此使用以下命令来安装`Flask-WTF`:
`pip install flask-wtf`
## 使用步骤
* 自定义表单类，继承自wtforms.Form类
* 定义需要验证的字段，字段名必须与模板中input表单的name值保持一致
* 在需要验证的字段上，指定好验证器。
* 在视图函数，定义`form = RegistFrom(request.form)`，再调用`form.validate()`判断

## 代码实现
zhiliao.py:
```py
from flask import Flask,request,render_template
from wtforms import Form,StringField
from wtforms.validators import Length,EqualTo
class RegistFrom(Form):
    #变量名必须与注册页面表单接收的变量名一致
    username = StringField(validators=[Length(min=3,max=10,message="用户名长度必须为3-10字符")])
    password = StringField(validators=[Length(min=6,max=10,message="用户名长度必须为3-10字符")])
    password_repeat = StringField(validators=[EqualTo("password",message="两次输入不一致")])

app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello wolrd!'

@app.route('/regist/',methods=['GET','POST'])
def regist():
    if request.method == 'GET':
        return render_template('regist.html')
    else:
        form = RegistFrom(request.form)
        if form.validate():
            return "success"
        else:
            print(form.errors())
            return "fail"
```


regist.html:
```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title>知了注册</title>
  </head>
  <body>
    <form class="" action="" method="post">
      <table>
        <tbody>
          <tr>
            <td>
              用户名
            </td>
            <td>
              <input type="text" name="username" value="请输入用户名">
            </td>
          </tr>
          <tr>
            <td>
              密码
            </td>
            <td>
              <input type="text" name="password" value="请输入密码">
            </td>
            <td>
              重复密码
            </td>
            <td>
              <input type="text" name="password_repeat" value="请输入密码">
            </td>
          </tr>
          <tr>
            <td></td>
            <td>
            <input type="submit" name="" value="立即注册">
            </td>
          </tr>
        </tbody>
      </table>
    </form>
  </body>
</html>
```

# WTForms常用验证器
## WTForms常用验证器
* Email：验证上传的数据是否为邮箱。
* EqualTo：验证上传的数据是否和另外一个字段相等，常用的就是密码和确认密码两个字段是否相等。
* InputRequired：原始数据的需要验证。如果不是特殊情况，应该使用InputRequired。
* Length：长度限制，有min和max两个值进行限制。
* NumberRange：数字的区间，有min和max两个值限制，如果处在这两个数字之间则满足。
* Regexp：自定义正则表达式。
* URL：必须要是URL的形式。
* UUID：验证UUID。

为把项目规范化，新建`forms.py`，将所有表单验证相关代码放入其中。
forms.py
```py
from wtforms import Form,StringField,IntegerField
from wtforms.validators import Length,EqualTo,Email,InputRequired,NumberRange,Regexp,URL,UUID

class RegistFrom(Form):
    username = StringField(validators=[Length(min=3,max=10,message="用户名长度必须为3-10字符")])
    email = StringField(validators=[Email()])
    addr = StringField(validators=[InputRequired()])
    age = IntegerField(validators=[NumberRange(min=12,max=80,message="12-80")])
    phone = StringField(validators=[Regexp(r'1[38745]\d{9}')])
    url = StringField(validators=[URL()])
    UUID = StringField(validators=[UUID()])
```

zhiliao.py
```py
@app.route('/login/',methods=['GET','POST'])
def login():
    if request.method == 'GET':
        return render_template('login.html')
    else:
        form = RegistFrom(request.form)
        if form.validate():
            return "success"
        else:
            return "fail"
```

# WTForms自定义验证验证字段
## 步骤
* 定义一个方法，方法名为`validate_字段名(self,field)`。
* 在方法中使用`field.data`可获取用户输入具体字符串数据。
* 如果验证成功，则pass，验证失败则抛出异常`raise ValidationError("错误提示内容")`
## 例子及代码实现
需自定义验证码验证功能，先假定验证码为1234时验证通过。
forms.py
```py
from wtforms import Form,StringField,IntegerField
from wtforms.validators import Length,EqualTo,Email,InputRequired,NumberRange,Regexp,URL,UUID,ValidationError

class RegistFrom(Form):
    # username = StringField(validators=[Length(min=3,max=10,message="用户名长度必须为3-10字符")])
    # email = StringField(validators=[Email()])
    # addr = StringField(validators=[InputRequired()])
    # age = IntegerField(validators=[NumberRange(min=12,max=80,message="12-80")])
    # phone = StringField(validators=[Regexp(r'1[38745]\d{9}')])
    # url = StringField(validators=[URL()])
    # UUID = StringField(validators=[UUID()])
    captcha = StringField(validators = [Length(min=4,max=4)])
    # 假设1234为验证成功
    def validate_captcha(self,field):
        '''
        函数命名必须为'validate'+'_'+'field'，这样在验证captcha时会自动将field=captcha传入验证自定义函数,另外，下文中field.data为实际用户输入的字符串。
        '''
        if field.data != '1234':
            raise ValidationError("验证码错误！")
```

# 使用WTForms渲染模板
## 渲染模板
forms.py
```py
class SettingsForm(Form):
    username = StringField("用户名：",validators=[InputRequired()])
    #第一个参数是lable，为username的指定名称
    tags = SelectedField('标签',choices=[('1','python'),('2','ios')])
```

zhiliao.py
```py
from forms import SettingsForm
@app.route("/settings/",methods = ['GET','POST'])
def settings():
    if request.method = 'GET':
        form = SettingsForm()
        return render_template('settings.html',form=form)
    else:
        pass
```

regist.html:
```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title>知了注册</title>
    <style media="screen">
      .username-input {
        background-color: red;
      }
    </style>
  </head>
  <body>
    <form class="" action="" method="post">
      <table>
        <tbody>
          <tr>
            <td>
              {{ form.username.lable }}
            </td>
            <td>
              {{ form.username(class='username-input') }}
            </td>
          </tr>
          <tr>
            <td>
              {{ form.tags.lable }}
            </td>
            <td>
              {{ form.tags() }}
            </td>
          </tr>
          <tr>
            <td></td>
            <td>
            <input type="submit" name="" value="立即注册">
            </td>
          </tr>
        </tbody>
      </table>
    </form>
  </body>
</html>
```

# 上传文件以及读取上传文件
## 上传文件的表单设置
upload.html
```html
<!DOCTYPE html>
<html lang="en" dir="ltr">
  <head>
    <meta charset="utf-8">
    <title></title>
  </head>
  <body>
      <form class="" action="" method="post" enctype="multipart/form-data">
        <table>
          <tbody>
            <tr>
              <td>头像：</td>
              <td>
                <input type="file" name="avatar">
              </td>
            </tr>
            <tr>
              <td>头像：</td>
              <td>
                <input type="text" name="desc">
              </td>
            </tr>
            <tr>
              <td></td>
              <td>
                <input type="submit" name="提交" value="">
              </td>
            </tr>
          </tbody>
        </table>
      </form>
  </body>
</html>
```
## 脚本接收设置
02_upload_file.py
```py
from flask import Flask,render_template,request
import os
from flask import send_from_directory
from werkzeug.utils import secure_filename
#命名上传文件的保存路径变量，意义为脚本文件目录下的images文件夹下。
UPLOAD_PATH = os.path.join(os.path.dirname(__file__),'images')
app = Flask(__name__)
@app.route('/')
def hello_world():
    pass
@app.route('/upload/',method=['GET','POST'])
def upload():
    if request.method == 'GET':
        return render_template('upload.html')
    else:
        des = request.form.get("desc")
        #获取文件
        avatar = request.files.get("avatar")
        #为防止命名重复或安全问题，用secure_filename函数处理以避免。
        filename = secure_filename(avatar.filename)
        #保存到当前文件夹下的images文件夹中
        avatar.save(os.path.join(UPLOAD_PATH, filename))
        return '文件上传成功！'

@app.route('/images/<filename>/')
def get_image(filename):
    #第一个参数为寻找路径，第二个为文件名
    return send_from_directory(UPLOAD_PATH, filename)
```
## 要点
* form的关键字设置`enctype="multipart/form-data"`
* input的关键字设置`type="file"`
* 后台需使用`request.file.get('表单中设置的文件变量名')`
* 后台获取文件后保存时需使用werkzeug.utils.secure_filename('文件名')处理文件名，以保证安全性
* 后台获取文件后保存:获取的文件.save(os.path.join(保存路径, 处理后的文件名))

# 使用flask_wtf验证上传的文件
## 步骤
* 定义表单时文件字段需要采用`FileField`这个类型
* 验证器应该由`flask_wtf.file`导入。`flask_wtf.file.FileRequired`用于验证是否为空，`flask_wtf.file.FileAllowed`用于验证是否是指定格式。
* 在后台获取文件和描述时，因各自的获取方法不同(`request.files.get()`和`request.form.get()`)，需将两者整合到同一个表单中以便验证，需调用`form = werkzeug.datastructures.CombinedMultiDict([request.files,request.form])`将两者整合，之后再`form.validate()`以进行验证。
## 代码实现
02_upload_file.py
```py
from wtforms import UploadForm
from werkzeug.datastructures import CombinedMultiDict
@app.route('/upload/',method=['GET','POST'])
def upload():
    if request.method == 'GET':
        return render_template('upload.html')
    else:
        form = UploadForm(CombinedMultiDict([request.files,request.form]))
        if form.validate():
            des = request.form.get("desc")
            #获取文件
            avatar = request.files.get("avatar")
            #为防止命名重复或安全问题，用secure_filename函数处理以避免。
            filename = secure_filename(avatar.filename)
            #保存到当前文件夹下的images文件夹中
            avatar.save(os.path.join(UPLOAD_PATH, filename))
            return '文件上传成功！'
        else:
            print(form.errors)
            return 'fail'
```

forms.py
```py
from wtforms import Form
from flask_wtf.file import FileRequired,FileAllowed
from wtforms.validators import InputRequired
class UploadForm(Form):
    #必须有上传文件，且必须为指定格式。
    avatar = FileField(validators=[FileRequired(),FileAllowed(['jpg','png','gif'])])
    #必须有描述
    desc = StringField(validators=[InputRequired()])
```
