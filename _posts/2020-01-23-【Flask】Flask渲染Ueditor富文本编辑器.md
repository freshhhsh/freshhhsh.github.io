---
published: true
title: Flask渲染Ueditor富文本编辑器实现图片视频上传、涂鸦功能
category: flask
tags:
  - python
  - flask
layout: post
---
# 教程原链接：
[Flask项目集成富文本编辑器UEditor 实现图片上传功能](http://flask123.sinaapp.com/article/47/)

# 本文实现功能
本文介绍如何在Flask项目中集成富文本编辑器UEditor，并实现文件上传、图片上传、视频上传及涂鸦功能。

# UEditor简介
[ueditor](http://ueditor.baidu.com/)是由百度「FEX前端研发团队」开发的所见即所得富文本web编辑器，具有轻量，可定制，注重用户体验等特点，开源基于MIT协议，允许自由使用和修改代码。

由于1.4.2版本之后的版本与之前版本存在较大的差异，本文以1.4.3版本为蓝本。

具体文档参见：[官方文档](http://fex-team.github.io/ueditor/)

# 在Flask项目中加入UEditor
## 下载UEditor:
访问[ueditor](http://ueditor.baidu.com/)首页，下载1.4.3 PHP UTF-8版本的UEditor，并解压到Flask应用程序的static目录。解压之后的目录结构是这样的：
```
| static/
| | ueditor/
| | |+dialogs/
| | |+lang/
| | |+php/
| | |+themes/
| | |+third-party/
| | |-config.json
| | |-index.html
| | |-ueditor.all.js
| | |-ueditor.all.min.js
| | |-ueditor.config.js
| | |-ueditor.parse.js
| | |-ueditor.parse.min.js
```
+表示目录。

## 在项目中加入UEditor：
我们在Flask应用程序的templates目录新建一个index.html文件（可根据实际情况选择文件名，或者把代码加入需要使用UEditor的文件）：

在head标签加入下面几行：
```html
<script type="text/javascript" charset="utf-8" src="{{ url_for('static', filename='ueditor/ueditor.config.js') }}"></script>
<script type="text/javascript" charset="utf-8" src="{{ url_for('static', filename='ueditor/ueditor.all.min.js') }}"> </script>
<!--建议手动加在语言，避免在ie下有时因为加载语言失败导致编辑器加载失败-->
<!--这里加载的语言文件会覆盖你在配置项目里添加的语言类型，比如你在配置项目里配置的是英文，这里加载的中文，那最后就是中文-->
<script type="text/javascript" charset="utf-8" src="{{ url_for('static', filename='ueditor/lang/zh-cn/zh-cn.js') }}"></script>
```

在body标签加入：
```html
<script id="editor" type="text/plain"></script>

<script type="text/javascript">
    //实例化编辑器
    //建议使用工厂方法getEditor创建和引用编辑器实例，如果在某个闭包下引用该编辑器，直接调用UE.getEditor('editor')就能拿到相关的实例
    var ue = UE.getEditor('editor', {
        serverUrl: "/upload/"
    });
</script>
```

## 请求路径配置：
UEditor 1.4.2+ 起，推荐使用统一的请求路径，在部署好前端代码后，需要修改 `ueditor.config.js` 里的 `serverUrl` 参数（或者初始化时指定，见上面的代码），改成 `'/upload/'` 。

`UEditor`初始化时，会向后端请求配置文件，后端收到请求后返回JSON格式的配置文件。具体实现参照后面的代码。

详细配置内容参见文档。

## 创建Flask应用程序（app.py）：
```py
# -*- coding: utf-8 -*-
# filename: app.py

from flask import Flask, render_template

app = Flask(__name__)

@app.route('/')
def index():
    return render_template('index.html')

@app.route('/upload/', methods=['GET', 'POST'])
def upload():
    pass

if __name__ == '__main__':
    app.run(debug=True)
```

应用程序运行之后，我们访问 http://localhost:5000/ 就可以看到UEditor编辑器了，但此时的上传图片、上传视频、涂鸦等按钮是暗色的，无法点击。

## UEditor后端请求规范说明
### 1、请求url
前端请求通过唯一的后台定义url `/upload/` 处理前端的请求
### 2、初始化get请求
在访问带有ueditor的前端页面时，前端会自动通过js向`/upload/`发送`GET`请求，发送的json数据为`{'action':'config'}`，后端接收到`get`请求后，验证键`'action'`对应的值为`'config'`后，则需要返回`json`数据的`config`对象给前端。为免去麻烦，此时返回的`config`对象直接引用`ueditor.config.js`的内容，该js文件内全部为相关的初始化配置键值对。
### 3、上传图片、视频、文件的get及post请求
当前端要上传图片、上传视频、上传文件时，对应get请求传给后端的json数据为`{'action':''uploadimage'}`或`{'action':'uploadvideo'}`或`{'action':'uploadfile'}'`。同时，也会post请求给后端发送前端上传的相应图片或视频或文件，即`{"upfile": file }`:

①通过`upload_file = request.files.get("upfile")`获取文件

②通过`filename = upload_file.filename`获取文件名。

### 4、涂鸦功能的get及post请求
当前端要涂鸦时，对应get请求传给后端的json数据为`{'action':'uploadscrawl'}'`。同时，也会post请求给后端发送前端上传的相应涂鸦的经过BASE64编码的图片，即`{"upfile": Base64Data }`，（一般为PNG格式），可通过后端收到之后解码的方式获取涂鸦图片:
```py
base64data = request.form['upfile']  # 这个表单名称以配置文件为准
img = base64.b64decode(base64data)
```

### 5、前端本地文件读取器
省去不必要的请求，去除涂鸦添加背景的请求，用前端FileReader读取本地图片代替
### 6、返回json数据格式
请求返回数据的格式，常规返回`json`字符串，数据包含：
1. `state`属性（成功时返回`'SUCCESS'`，错误时返回错误信息）。
2. `url`属性:（后端获取前端post请求传入的文件后，进行相应的处理，可保存在本地路径也可上传至七牛云存储等，之后返回本地文件的url或云存储的文件url，以供前端通过访问url显示图片）
3. `title`属性：文件的名称
4. `original`: 文件的源名称

总之，返回的json数据对象为：
```json
{
  "state": "SUCCESS" ,
  "url": "保存后的文件的url",
  "title": "文件名",
  "original": "源文件名"
}
```
### 7、 jsonp请求格式
请求支持jsonp请求格式，当请求有通过GET方式传callback的参数时，返回json数据前后加上括号，再在前面加上callback的值，格式类似这样： cb({"key": "value"})。

## 主要功能代码
### 获取配置信息
```python
@app.route('/upload/', methods=['GET', 'POST'])
def upload():
    action = request.args.get('action')

    # 解析JSON格式的配置文件
    # 这里使用PHP版本自带的config.json文件
    with open(os.path.join(app.static_folder, 'ueditor', 'php',
                           'config.json')) as fp:
        try:
            # 删除 `/**/` 之间的注释
            CONFIG = json.loads(re.sub(r'\/\*.*\*\/', '', fp.read()))
        except:
            CONFIG = {}

    if action == 'config':
        # 初始化时，返回配置文件给客户端
        result = CONFIG

    return json.dumps(result)
```

### 文件、视频、图片上传实现
```python
@app.route('/upload/', methods=['GET', 'POST'])
def upload():
    result = {}
    action = request.args.get('action')

    if action in ('uploadimage', 'uploadvideo', 'uploadfile'):
        upfile = request.files['upfile']  # 这个表单名称以配置文件为准
        # upfile 为 FileStorage 对象
        # 这里保存文件并返回相应的URL
        upfile.save(filename_to_save)
        result = {
            "state": "SUCCESS",
            "url": "upload/demo.jpg",
            "title": "demo.jpg",
            "original": "demo.jpg"
        }

    return json.dumps(result)
```

### 涂鸦功能
```py
@app.route('/upload/', methods=['GET', 'POST'])
def upload():
    result = {}
    action = request.args.get('action')

    if action in ('uploadscrawl'):
        base64data = request.form['upfile']  # 这个表单名称以配置文件为准
        img = base64.b64decode(base64data)
        # 这里保存文件并返回相应的URL
        with open(filename_to_save, 'wb') as fp:
            fp.write(img)
        result = {
            "state": "SUCCESS",
            "url": "upload/demo.jpg",
            "title": "demo.jpg",
            "original": "demo.jpg"
        }

    return json.dumps(result)
```

## 完整示例
此为知了黄勇老师自制的插件，运用flask渲染实现Ueditor的七牛云存储/本地存储图片、视频、文件及涂鸦功能（通过蓝图实现可组装）
```py
#encoding: utf-8

from flask import (
    Blueprint,
    request,
    jsonify,
    url_for,
    send_from_directory,
    current_app as app
)
import json
import re
import string
import time
import hashlib
import random
import base64
import sys
import os
from urllib import parse
# 更改工作目录。这么做的目的是七牛qiniu的sdk
# 在设置缓存路径的时候默认会设置到C:/Windows/System32下面
# 会造成没有权限创建。
os.chdir(os.path.abspath(sys.path[0]))
try:
    import qiniu
except:
    pass
from io import BytesIO

bp = Blueprint('ueditor',__name__,url_prefix='/ueditor')

UEDITOR_UPLOAD_PATH = ""
UEDITOR_UPLOAD_TO_QINIU = False
UEDITOR_QINIU_ACCESS_KEY = ""
UEDITOR_QINIU_SECRET_KEY = ""
UEDITOR_QINIU_BUCKET_NAME = ""
UEDITOR_QINIU_DOMAIN = ""

@bp.before_app_first_request
def before_first_request():
    global UEDITOR_UPLOAD_PATH
    global UEDITOR_UPLOAD_TO_QINIU
    global UEDITOR_QINIU_ACCESS_KEY
    global UEDITOR_QINIU_SECRET_KEY
    global UEDITOR_QINIU_BUCKET_NAME
    global UEDITOR_QINIU_DOMAIN
    UEDITOR_UPLOAD_PATH = app.config.get('UEDITOR_UPLOAD_PATH')
    if UEDITOR_UPLOAD_PATH and not os.path.exists(UEDITOR_UPLOAD_PATH):
        os.mkdir(UEDITOR_UPLOAD_PATH)

    UEDITOR_UPLOAD_TO_QINIU = app.config.get("UEDITOR_UPLOAD_TO_QINIU")
    if UEDITOR_UPLOAD_TO_QINIU:
        try:
            UEDITOR_QINIU_ACCESS_KEY = app.config["UEDITOR_QINIU_ACCESS_KEY"]
            UEDITOR_QINIU_SECRET_KEY = app.config["UEDITOR_QINIU_SECRET_KEY"]
            UEDITOR_QINIU_BUCKET_NAME = app.config["UEDITOR_QINIU_BUCKET_NAME"]
            UEDITOR_QINIU_DOMAIN = app.config["UEDITOR_QINIU_DOMAIN"]
        except Exception as e:
            option = e.args[0]
            raise RuntimeError('请在app.config中配置%s！'%option)

    csrf = app.extensions.get('csrf')
    if csrf:
        csrf.exempt(upload)


def _random_filename(rawfilename):
    letters = string.ascii_letters
    random_filename = str(time.time()) + "".join(random.sample(letters,5))
    filename = hashlib.md5(random_filename.encode('utf-8')).hexdigest()
    subffix = os.path.splitext(rawfilename)[-1]
    return filename + subffix


@bp.route('/upload/',methods=['GET','POST'])
def upload():
    action = request.args.get('action')
    result = {}
    if action == 'config':
        config_path = os.path.join(bp.static_folder or app.static_folder,'ueditor','config.json')
        with open(config_path,'r',encoding='utf-8') as fp:
            result = json.loads(re.sub(r'\/\*.*\*\/','',fp.read()))

    elif action in ['uploadimage','uploadvideo','uploadfile']:
        image = request.files.get("upfile")
        filename = image.filename
        save_filename = _random_filename(filename)
        result = {
            'state': '',
            'url': '',
            'title': '',
            'original': ''
        }
        if UEDITOR_UPLOAD_TO_QINIU:
            if not sys.modules.get('qiniu'):
                raise RuntimeError('没有导入qiniu模块！')
            buffer = BytesIO()
            image.save(buffer)
            buffer.seek(0)
            q = qiniu.Auth(UEDITOR_QINIU_ACCESS_KEY, UEDITOR_QINIU_SECRET_KEY)
            token = q.upload_token(UEDITOR_QINIU_BUCKET_NAME)
            ret,info = qiniu.put_data(token,save_filename,buffer.read())
            if info.ok:
                result['state'] = "SUCCESS"
                result['url'] = parse.urljoin(UEDITOR_QINIU_DOMAIN,ret['key'])
                result['title'] = ret['key']
                result['original'] = ret['key']
        else:
            image.save(os.path.join(UEDITOR_UPLOAD_PATH, save_filename))
            result['state'] = "SUCCESS"
            result['url'] = url_for('ueditor.files',filename=save_filename)
            result['title'] = save_filename,
            result['original'] = image.filename

    elif action == 'uploadscrawl':
        base64data = request.form.get("upfile")
        img = base64.b64decode(base64data)
        filename = _random_filename('xx.png')
        filepath = os.path.join(UEDITOR_UPLOAD_PATH,filename)
        with open(filepath,'wb') as fp:
            fp.write(img)
        result = {
            "state": "SUCCESS",
            "url": url_for('files',filename=filename),
            "title": filename,
            "original": filename
        }
    return jsonify(result)


@bp.route('/files/<filename>/')
def files(filename):
    return send_from_directory(UEDITOR_UPLOAD_PATH,filename)
```
