---
published: true
title: 【Flask】Flask-paginate分页
category: flask
tags:
  - python
  - flask
layout: post
---
# Flask-paginate包分页功能使用示例
源官方文档：https://pythonhosted.org/Flask-paginate/
`pip install flask-paginate`
蓝图文件中：
```python
from flask import Blueprint
from flask_paginate import Pagination, get_page_parameter

mod = Blueprint('users', __name__)


@mod.route('/')
def index():
    search = False
    q = request.args.get('q')
    if q:
        search = True

    #从url中获取第几页，get_page_parameter()会自动获取所在第几页，若未获取则默认为1.
    page = request.args.get(get_page_parameter(), type=int, default=1)

    #users为User表中需要查询的数据
    users = User.find(...)

    pagination = Pagination(page=page, total=users.count(), search=search, record_name='users')
    # 'page' is the default name of the page parameter, it can be customized
    # e.g. Pagination(page_parameter='p', ...)
    # or set PAGE_PARAMETER in config file
    # also likes page_parameter, you can customize for per_page_parameter
    # you can set PER_PAGE_PARAMETER in config file
    # e.g. Pagination(per_page_parameter='pp')

    return render_template('users/index.html',
                           users=users,
                           pagination=pagination,
                           )
```


```html
/*一页数据显示的内容*/
<table>
  <thead>
    <tr>
      <th>#</th>
      <th>Name</th>
      <th>Email</th>
    </tr>
  </thead>
  <tbody>
    {% for user in users %}
      <tr>
        <td>{{ loop.index + pagination.skip }}</td>
        <td>{{ user.name }}</td>
        <td>{{ user.email }}</td>
      </tr>
    {% endfor %}
  </tbody>
</table>

/*此为选择页面的选择框*/
{{ pagination.links }}
```
## 初始化时可选择参数
found: used when searching

page: current page

per_page: how many records displayed on one page

page_parameter: a name(string) of a GET parameter that holds a page index. Use it if you want to iterate over multiple Pagination objects simultaniously. defautl is ‘page’.

per_page_parameter: a name for per_page likes page_parameter. default is ‘per_page’.

inner_window: how many links arround current page

outer_window: how many links near first/last link

prev_label: text for previous page, default is ‘&laquo;’

next_label: text for next page, default is ‘&raquo;’

search: search or not?

total: total records for pagination

display_msg: text for pagation information

search_msg: text for search information

record_name: record name showed in pagination information

link_size: font size of page links

alignment: the alignment of pagination links

href: Add custom href for links - this supports forms with post method. MUST contain {0} to format page number

show_single_page: decide whether or not a single page returns pagination

bs_version: the version of bootstrap, default is 2

css_framework: the css framework, default is ‘bootstrap’

anchor: anchor parameter, appends to page href

format_total: number format total, like 1,234, default is False

format_number: number format start and end, like 1,234, default is False

## 项目示例
如：
```python
@bp.route('/')
def index():
    PER_PAGE = 20
    page = request.args.get(get_page_parameter(), type=int, default=1)
    start = (page-1) * config.PER_PAGE
    end = start + config.PER_PAGE
    posts = PostModel.query.slice(start,end)
    pagination = Pagination(page=page, total=users.count(), search=search,record_name='users',outer_window=0,inner_window=2,per_page=20)
    context = {
      "banners": banners,
      "boards": boards,
      "posts": posts,
      "pagination": pagination
    }
    return render_template("front/front_index.html",**context)
```

```html
<ul class="post-list-group">
    {% for post in posts %}
        <li class="post-info">
            <div class="post-avatar">
                <img src="{{ post.author.avatar or url_for('static',filename='front/imgs/default_avatar.jpg') }}" alt="">
            </div>
            <div class="post-info-group">
                <p class="post-title"><a href="#">{{ post.title }}</a></p>
                <p class="post-info">
                    <span>作者：<a href="#">{{ post.author.username }}</a></span>
                    <span>发布时间：{{ post.create_time }}</span>
                    <span>评论：0</span>
                    <span>点赞：0</span>
                </p>
            </div>
        </li>
    {% endfor %}
</ul>
<div style="text-align: center">
    {{ pagination.links }}
</div>
```
