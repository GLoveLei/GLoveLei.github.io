---
title: 'flask SQLAlchemy 的安装和基础应用'
date: '2018/12/23 14:59:21'
categories:
   - flask
tags:
   - flask
toc_level: 3
version: 3
---

SQLAlchemy是一个基于Python实现的ORM框架。该框架建立在 DB API之上，使用关系对象映射进行数据库操作，简言之便是：将类和对象转换成SQL，然后使用数据API执行SQL并获取执行结果。

### 安装命令

```
pip install flask-sqlalchemy
```

注意sqlalchemy 依赖于 pymysql 模块，确保pymysql 被正确安装

```
pip install pymysql
```

### 建立对象

```
#导入第三方连接库sql点金术
from flask_sqlalchemy import SQLAlchemy

#建立对象
app = Flask(__name__)

#载入配置文件
app.config.from_pyfile('config.ini')

# #指定数据库连接还有库名
# app.config['SQLALCHEMY_DATABASE_URI'] = 'mysql+pymysql://root:123456@127.0.0.1:3306/myflask?charset=utf8'

#指定配置，用来省略提交操作
#app.config['SQLALCHEMY_COMMIT_ON_TEARDOWN'] = True

#建立数据库对象
db = SQLAlchemy(app) 

#建立数据库类，用来映射数据库表,将数据库的模型作为参数传入
class User(db.Model):
    #声明表名
    __tablename__ = 'user'
    #建立字段函数
    id = db.Column(db.Integer,primary_key=True)
    name = db.Column(db.String(200))
    password = db.Column(db.String(200))
```

  

