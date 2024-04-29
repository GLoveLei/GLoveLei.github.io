---
title: 'celery异步框架'
date: '2021/1/30 14:59:21'
categories:
   - python
tags:
   - celery异步框架
toc_level: 3
version: 3
---
![cover](images/river.png)

## 关于Celery
    Celery 是一款非常简单、灵活、可靠的分布式系统，
    可用于处理大量消息，并且提供了一整套操作此系统的一系列工具，
    同时Celery 是一款消息队列工具，可用于处理实时数据以及任务调度。
    
#### 一、安装
    pip install celery

#### 二、注册创建任务
```js
from celery import Celery
import time
redis=Celery('hello', broker='redis://127.0.0.1:6379', backend='redis://127.0.0.1:6379')

@redis.task
def sendto():
    print("hello gaolei")
    time.sleep(5)
    return "发送成功"
    
if __name__ == '__main__':
    res=sendto.delay()
    print(res)
```
task共享

用shared_task修饰时，不需要依赖于指定实例对象。
```js
from celery import shared_task
@shared_task
def foo():
    return "hello world"
    
if __name__ == '__main__':
    res=foo.delay()
    print(res)
```
#### 三、启动Worker
启动Worker，监听Broker中是否有任务，

    celery -A tasks worker --loglevel=info

#### 四、调用任务
在主程序中调用任务，掉任务发送给 Broker， 而不是真正执行该任务
```js
from tasks import sendto
import time


def register():
    start = time.time()
    print("插入记录到数据库")
    print("2. celery发送hello gaolei")
    sendto.delay("hello")
    print("注册成功")
    print("耗时：%s 秒 " % (time.time() - start))

if __name__ == '__main__':
    register()

```
执行任务花费5秒

#### 五、总结

    celery的设计于很多模块相耦合，比如broker可以使用redis来进行操作
    相对于保存结果而言，celery需要在某个地方存储或发送状态，支持内置后端。
    程序运行过程中，要执行耗时的任务，不想主程序被阻塞，常见的方法是多线程。但是多线程并发过量也需要限制并发个数所以使用celery。
    