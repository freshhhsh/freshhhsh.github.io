---
published: true
title: 【Flask】第十二章 CSRF攻击
category: flask_MySQL
tags:
  - python
  - flask
layout: post
---
# CSRF攻击原理
## CSRF攻击介绍
CSRF（Cross Site Request Forgery, 跨站域请求伪造）是一种网络的攻击方式，它在 2007 年曾被列为互联网 20 大安全隐患之一。其他安全隐患，比如 SQL 脚本注入，跨站域脚本攻击等在近年来已经逐渐为众人熟知，很多网站也都针对他们进行了防御。然而，对于大多数人来说，CSRF 却依然是一个陌生的概念。即便是大名鼎鼎的 Gmail, 在 2007 年底也存在着 CSRF 漏洞，从而被黑客攻击而使 Gmail 的用户造成巨大的损失。
## CSRF攻击原理
网站是通过cookie来实现登录功能的。而cookie只要存在浏览器中，那么浏览器在访问这个cookie的服务器的时候，就会自动的携带cookie信息到服务器上去。那么这时候就存在一个漏洞了，如果你访问了一个别有用心或病毒网站，这个网站可以在网页源代码中插入js代码，使用js代码给其他服务器发送请求（比如ICBC的转账请求）。那么因为在发送请求的时候，浏览器会自动的把cookie发送给对应的服务器，这时候相应的服务器（比如ICBC网站），就不知道这个请求是伪造的，就被欺骗过去了。从而达到在用户不知情的情况下，给某个服务器发送了一个请求（比如转账）。
# CSRF防御原理
当客户访问官网时，官网返回一个含有服务器随机生成的加密`csrf_token`的cookie。之后，用户访问官网表单页面时，表单有一项`<input type="hidden" name="csrf_token" value="与cookie中相同的服务器随机生成值（但加密后不一样，解密后相同）"`。当用户提交后，如果提交的表单中`csrf_token`与cookie中保存的`csrf_token`值相同，则说明是正常方式访问及操作。

# Flask中CSRF防御的方法与原理
flask_wtf包中已提供CSRF防御方法。
进行CSRF防御只需要两步：
* 在cookie中添加csrf_token，主脚本实现
* 在表单中添加csrf_token，html模板实现

### 在cookie中添加csrf_token
```py
from flask_wtf import CSRFProtect
CSRFProtect(app)
```
### 在表单中添加csrf_token
```html
<form class="" action="index.html" method="post">
  <input type="hidden" name="csrf_token" value="{{ csrf_token() }}">
  <table>
    ...
  </table>
</form>
```

# AJAX处理CSRF漏洞
## AJAX
Ajax 即“Asynchronous Javascript And XML”（异步 JavaScript 和 XML），是指一种创建交互式网页应用的网页开发技术。通过在后台与服务器进行少量数据交换，Ajax 可以使网页实现异步更新。这意味着可以在不重新加载整个网页的情况下，对网页的某部分进行更新。

##具体防御方法
在`login.html`中:
```html
<head>
    /*在网页头信息中加入csrf_token，然后把负责csrf的input标签删除。*/
    <meta name="csrf_token" content="{{ csrf_token() }}">
    <script src="https://cdn.bootcss.com/jquery/3.4.1/jquery.min.js"></script>
    <script src="{{ urf_for('static',filename='login.js') }}"></script>
</head>

...
<td><input type="submit" value="立即登录" id="submit"></td>
```

在static文件夹下：
zlajax.js
```js
// 对jquery的ajax的封装

'use strict';
var zlajax = {
	'get':function(args) {
		args['method'] = 'get';
		this.ajax(args);
	},
	'post':function(args) {
		args['method'] = 'post';
		this.ajax(args);
	},
	'ajax':function(args) {
		// 设置csrftoken
		this._ajaxSetup();
		$.ajax(args);
	},
  //提交时在请求头中放入csrftoken
	'_ajaxSetup': function() {
		$.ajaxSetup({
			'beforeSend':function(xhr,settings) {
				if (!/^(GET|HEAD|OPTIONS|TRACE)$/i.test(settings.type) && !this.crossDomain) {
                    var csrftoken = $('meta[name=csrf-token]').attr('content');
                    xhr.setRequestHeader("X-CSRFToken", csrftoken)
                }
			}
		});
	}
};
```

login.js
```js
$(function() {
    $('#submit').click(function(event) {
        //阻止点击后默认提交表单的行为
        event.preventDefault();
        //提交时获取input标签的值
        var email = $('input[name=email]').val();
        var password = $('input[name=password]').val();
        })

        zlajax.post({
            'url': '/login/',
            'data': {
              'email':email,
              'password':password
            },
            'success': function(data) {
                console.log(data);
            },
            'fail': function(error) {
                console.log(error);
            }
        });
    });
  });
```
