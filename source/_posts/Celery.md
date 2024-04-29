---
title: 'Celery'
date: '2021/3/21 14:59:21'
categories:
   - python
tags:
   - Celery来实现异步任务队列以及定时任务
toc_level: 3
version: 3
---
![cover](images/tree.png)
一、安装celery
    pip install celery


二、创建一个任务文件task.py
```js
from celery.task import task

# 自定义要执行的task任务
@task
def print_test():
	print("gaolei")
	return 'hello celery'
```
三、配置settings.py文件
```js
CELERY_BROKER_URL = 'redis://localhost:6379/'

CELERY_RESULT_BACKEND = 'redis://localhost:6379/'

CELERY_RESULT_SERIALIZER = 'json'
```
四、在settings.py同级目录创建celery.py
```js
from __future__ import absolute_import, unicode_literals
import os
from celery import Celery

# 设置环境变量
os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'mydjango.settings')

# 注册Celery的APP
app = Celery('mydjango')
# 绑定配置文件
app.config_from_object('django.conf:settings', namespace='CELERY')

# 自动发现各个app下的tasks.py文件
app.autodiscover_tasks()
```
五、修改settings.py同级目录的init.py文件
```js
from __future__ import absolute_import, unicode_literals
from .celery import app as celery_app

#导包
import pymysql
#初始化
pymysql.install_as_MySQLdb()



__all__ = ['celery_app']
```
六、通过view进行在线调用
```js
from user import tasks

def ctest(request,*args,**kwargs):
    res=tasks.print_test.delay()
    #任务逻辑
    return JsonResponse({'status':'successful','task_id':res.task_id})
```
七、在manage.py的目录下启动celery服务

    celery worker -A mydjango -l info -P eventlet