---
published: true
title: 【Flask数据库】filter方法常用过滤条件
category: flask_MySQL
tags:
  - python
  - flask
  - MySQL
layout: post
---
# 一、过滤条件
过滤是数据提取的重要功能，以下为filter方法实现的过滤:
******
### 1、 equals：等于
`session.query.filter(User.name == 'ed')`
*****
### 2、 not equals：不等于
`session.query.filter(User.name != 'ed')`
*****
### 3、 like：为Column的一个方法，表模糊匹配，like内传入匹配字段。
`session.query.filter(User.name.like('%ed%'))`
注：'%'为sql的通配符，匹配一个或多个字符。
        '_'仅替代一个字符。
        [abc]匹配abc中任意一个。
        [^abc]/[!abc]匹配非abc的任意字符。

注：ilike与like用法相同，但ilike不区分大小写
******
### 4、 in_：判度胺是否存在
`session.query.filter(User.name.in_(['xushenghai','zhiliao']))`
*****
### 5、 not in:判断是否不存在
`session.query.filter(~User.name.in_(['xushenghai','zhiliao']))` #波浪表取反
或：
`session.query.filter(User.name.notin_(['xushenghai','zhiliao']))`
***
### 6、 is null:为空
先给数据库的article表添加一个空列content：
`alter table article add column content text`
再在脚本中加入content的Column对象
筛选是否为空:
`articles = session.query(Article).filter(Article.content == None).all()`
****
### 7、 is not null：不为空
`articles = session.query(Article).filter(Article.content != None).all()`
****
### 8、and：并，可用逗号表示并,也可通过导入and_
`from sqlalchemy import and_`
`articles = session.query(Article).filter(and_(Article.content == 'abc', Article.title == 'abc')).all()`
或不用and_：
`articles = session.query(Article).filter(Article.content == 'abc', Article.title == 'abc').all()`
****
### 9、 or：或
`from sqlalchemy import or_`
`articles = session.query(Article).filter(or_(Article.content == 'abc', Article.title == 'abc')).all()`
****
# 备注
要想直接看orm底层转换的sql的命令语句，可以在filter方法后面不再调用.all()方法，直接打印结果即可。
```
articles = session.query(Article).filter(or_(Article.content == 'abc', Article.title == 'abc'))
print(articles)
```
