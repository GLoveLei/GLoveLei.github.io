---
title: '三方支付'
date: '2021/1/29 14:59:21'
categories:
   - python
tags:
   - 三方支付
toc_level: 3
version: 3
---
![cover](images/river.png)

## 三方支付

1、注册一个沙箱号：https://open.alipay.com/platform/home.htm

####  一、安装插件
 ```js
pip install python-alipay-sdk --upgrade
```


#### 二、在settings中配置
```js
ALIPAY_APPID = ''
ALIPAY_URL = ""
ALIPAY_DEBUG = True
APP_PRIVATE_KEY_PATH = ''
ALIPAY_PUBLIC_KEY_PATH = ''
ALIPAY_GATE = 'https://openapi.alipaydev.com/gateway.do?'

```
#### 三、发起支付
请求方式：Get
```js
from rest_framework.views import APIView
from rest_framework.response import Response
from orders.models import *
from rest_framework import status
from alipay import AliPay  #支付
from mall import settings


class AliPayURLView(APIView):
    def get(self, request, order_id):
        # 根据order_id查询订单对象
        try:
            order_obj = OrderInfo.objects.get(pk=order_id)
        except:
            raise Exception("订单号无效")

        # 创建alipay对象
        alipay = AliPay(
            appid=settings.ALIPAY_APPID,
            app_notify_url=None,
            app_private_key_path=settings.ALIPAY_PRIVATE_KEY_PATH,
            alipay_public_key_path=settings.ALIPAY_PUBLIC_KEY_PATH,
            debug=settings.ALIPAY_DEBUG
        )
    subject='支付宝支付'
        # 跳转到 https://openapi.alipay.com/gateway.do? + order_string
    order_string = alipay.api_alipay_trade_page_pay(
        out_trade_no=order_id,
        total_amount=str(order.total_amount), # 这里类型我们要由 decimal转换为 str
        subject=subject,
        return_url="http://127.0.0.1/pay_success.html"
    )

    #  拼接url,并且返回
    url = settings.ALIPAY_URL + '?' + order_string

    return Response({'alipay_url':url})
```

#### 五、保存支付结果
请求方式： PUT
路径：/pay/?参数
```js
 class PayStatusAPIView(APIView):
    def put(self,request):
    # 1. 创建alipay对象
    app_private_key_string = settings.APP_PRIVATE_KEY_PATH
    alipay_public_key_string = settings.ALIPAY_PUBLIC_KEY_PATH

    alipay = AliPay(
        appid=settings.ALIPAY_APPID,
        app_notify_url=None,  # 回调url
        app_private_key_string=app_private_key_string,
        # 支付宝的公钥，验证支付宝回传消息使用
        alipay_public_key_string=alipay_public_key_string,
        sign_type="RSA2",  
        debug=settings.ALIPAY_DEBUG  # 默认False
    )

    # 2.验证数据
    data = request.query_params.dict()
    # sign 不能参与签名验证
    signature = data.pop("sign")


    # verify确认是否成功
    success = alipay.verify(data, signature)
    if success:
        # 3.验证成功之后,可以从 data中获取 支付宝的订单id和 我们的订单id
        # 支付宝的交易id
        trade_id = data.get('trade_id')
        # 商家id
        out_trade_id = data.get('out_trade_id')
        #1. 把支付宝的订单id和 我们的订单id 保存起来
        Payment.objects.create(
           order_id=out_trade_id,
            trade_id=trade_id
        )
        # 2. 更新订单的状态
        OrderInfo.objects.filter(order_id=out_trade_no).update(status=OrderInfo.ORDER_STATUS_ENUM['UNSEND'])
        #3. 返回 支付宝的订单id
        return Response({'trade_id':trade_id})
```
### 总结
    先创建沙箱账号，支付宝登录获取ALIPAY_APPID等信息，
    保存在settings.py文件下然后进行视图的编写，
    判断是否有下单时的订单号然后进行获取回调地址等信息，
    最后回调返回支付成功保存数据完成支付接口