---
title: 'Redis订阅与发布'
date: '2021/2/1 14:59:21'
categories:
   - redis
tags:
   - Redis订阅与发布
toc_level: 3
version: 3
---
![cover](images/river.png)


#### 一、什么是Redis
Redis是一个开源的内存数据库，它以键值对的形式存储数据。由于数据存储在内存中，因此Redis的速度很快，但是每次重启Redis服务时，其中的数据也会丢失，因此，Redis也提供了持久化存储机制(AOF,RDB)，将数据以某种形式保存在文件中，每次重启时，可以自动从文件加载数据到内存当中。 

#### 二、什么是发布订阅 
Redis 发布订阅是一种消息通信模式：发送者发送消息，订阅者接收消息。


#### 三、发布者
发布者是用来发布消息

publish 将信息发送到指定的频道

语法:
    
    publish channel message
 返回结果:
 
    接收到信息的订阅者数量
```js
import time
import redis
 
number = ['2000', '2001', '2002', '2003']
sing = ['12', '30', '1', '30']
 
redis = redis.Redis(host='127.0.0.1', port='6379', db=0, password='密码即可')
for i in range(len(number)):
    value = str(number[i]) + ' ' + str(sing[i])
    redis.publish("ceshi", value)  #发布消息到ceshi
```

#### 三、订阅者
```js
import time
import redis
 
redis = redis.Redis(host='127.0.0.1', port='6379', db=1, password='密码即可')
ps = redis.pubsub()
ps.subscribe('liao')  #从liao订阅消息
for item in ps.listen():        #监听状态：有消息发布了就拿过来
    if i['type'] == 'message':
        print i['change']
        print i['data']
```


#### 四、总结
①连接方式使用python连接redis
②是订阅方法。这里使用的是Redis类中的pubsub方法。连接好之后，可使用subscribe方法来订阅redis消息。其中subscribe是订阅一个频道，之后就可以开始监听了。