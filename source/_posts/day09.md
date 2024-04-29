---
title: 'redis键空间事件'
date: '2021/2/1 14:59:21'
categories:
   - redis
tags:
   - redis键空间事件
toc_level: 3
version: 3
---
![cover](images/river.png)


## Redis键空间介绍
使用场景:

在购物项目中，如果完成提交订单而没有支付成功的状态时，redis会在规定的时间内删除订单，redis保存订单后，设置过期时间，时间过后数据删除

促销活动结束之后，促销活动开始之前设置redis,到期删除，通过回调函数拿到key, 执行回调库存脚本


#### 一、开启键空间通知
在cmd窗口，默认键空间通知是关闭的,需要在终端开启

    redis-cli config set notify-keyspace-events KEA
    ok
key值可能让事件被启用

|字符|发送的通知	|
|:---:|:---|
|K|键空间通知，所有通知以__keyspace@<db>__为前缀|
|E|键空间通知，所有通知以__keyevent@<db>__为前缀|
|g|DEL、EXPIRE、RENAME等类型无关的通用命令通知|
|$|字符串命令通知|
|l|列表命令通知|
|s|结合命令通知|
|h|哈希命令通知|
|z|有序集合命令通知|
|x|过期事件:每当有过期键被删除时发送|
|e|驱逐(evict)事件:每当有键因为maxmemory政策而被删除时发送|
|A|参数g$lshzxe的别名|


#### 二、检查事件是否正常
    1 redis-cli --csv psubscribe '*'  
    2 # 结果   
    3 Reading messages... (press Ctrl-C to quit)  
    4 "psubscribe","*",1

#### 三、配置回调函数
在python的视图类中进行配置
```js
# 连接redis数据库
redis = Redis(host='127.0.0.1', port=6379, decode_responses=True)

# 监听新消息
pubsub = redis.pubsub()

# 定义触发事件
def orderbuy(msg):
    order_id = str(msg['data'])

    # 获取订单对象
    order = Order.objects.get(order_id=order_id)

    # 判断用户是否已经付款
    if str(order.status) == "1":
        # 取消订单,更改订单状态
        Order.objects.filter(order_id=order_id).update(status="2")
        # 获取订单中的所有商品
        goods = order.objects.all()
		# 遍历商品
        for good in goods:
            # 获取订单中的商品数量
            count = good.count
            print(count)
            # 获取name商品
            name = good.name
            # 将库存重新增加到sku的stock中去
            name.stock += count
            # 从销量中减去已经取消的数量
            name.sales -= count
            name.save()
#订阅redis键空间通知
pubsub.psubscribe(**{'__keyevent@0__:expired': event_handler})

# 死循环,接收订阅的通知
while True:
    message = pubsub.get_message()
    if message:
        print(message)
    else:
        time.sleep(0.1)
```

    keyevent@0:expired’:
    设定通知类型: keyspace、keyevent
    设置事件类型: expired
    选中数据库号: @0()

#### 总结
在购物车项目中进行添加，先在终端开启redis的键空间，然后开启键空间，在新的终端打开redis进行创建数据，就会有数据发送给键空间，然后保存，最后在python的视图函数中进行配置redis监听数据，判断是否支持，再发送数据给键空间，进行无线循环获取订阅获取的通知。