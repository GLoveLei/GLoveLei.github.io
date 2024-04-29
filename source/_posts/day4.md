---
title: '微博三方登录'
date: '2021/1/28 14:59:21'
categories:
   - python
tags:
   - 微博三方登录
toc_level: 3
version: 3
---
![cover](images/river.png)

## 总体流程
首先需要注册"新浪微博开放平台"：https://open.weibo.com

添加网站应用前需要先提交认证审核，然后添加完网站应用再提交应用审核。

创建完应用后会有一个 App Key 和 App Secret。

##### 1.获取 code
第三方客户端引导用户发送get请求

    https://api.weibo.com/oauth2/authorize
参数：

|参数名|是否必须	|含义|
|:---:|:---:|:---|
|client_id|是|申请应用时分配的AppKey。|
|redirect_uri|是|授权回调地址。|
|scope|否|申请scope权限所需参数，可一次申请多个scope权限，用逗号分隔|
|state|否|用于保持请求和回调的状态，在回调时，会在Query Parameter中回传该参数|
|display|否|授权页面的终端类型，取值见下面的说明。|
|forcelogin|否|是否强制用户重新登录，true：是，false：否。默认false。|
|language|否|授权页语言，缺省为中文简体版，en为英文版。|

这时会跳转到登入授权页面，授权后新浪微博会向客户端返回到我们的创建应用的时候填写的那个回调地址。并且带着code参数

##### 2.通过 code 获取 access_token
有了code，我们在通过 `App Key` 、`App Secret` 和`code`等参数去发送 post 请求，获取 `access_token`

    https://graph.qq.com/oauth2.0/token
###### 注意，虽然是post请求，但是参数是通过get传递的 参数：

|参数名|是否必须	|含义|
|:---:|:---:|:---|
|client_id|是|申请应用时分配的AppKey。|
|client_secret|是|申请应用时分配的AppSecret。|
|grant_type|是|请求的类型，填写authorization_code|
|code|是|调用authorize获得的code值。|
|redirect_uri|是|回调地址，需需与注册应用里的回调地址一致。|

###### 返回数据包括 access_token 、uid等信息：
```js
 {
       "access_token": "",
       "expires_in": '',
       "remind_in":"",
       "uid":""
 }
```

##### 3.用 access_token 获取用户信息
get请求地址：

    https://api.weibo.com/2/users/show.json

参数：

|参数名|是否必须	|含义|
|:---:|:---:|:---|
|access_token|是|采用OAuth授权方式为必填参数，OAuth授权后获得。|
|uid|否|需要查询的用户ID。|
|screen_name|否|需要查询的用户昵称。|

注意：参数uid与screen_name二者必选其一，且只能选其一。

我们可以用返回的 access_token 与 uid 访问获取用户信息

## 代码部分
#### 后端部分
首先在settings中配置
```js
# 微博第三方登录配置
App_Key = ''  # App Key
App_Secret = ''  # App Secret
MicroBlog_URL = ''  # 回调页地址
```
在views.py中进行登录配置操作

```js
import requests
from BlogDjango.settings import App_Key, App_Secret, MicroBlog_URL
from rest_framework.views import APIView
from rest_framework.response import Response
from django.shortcuts import redirect


class MicroBlogView(APIView):
    # 使用微博开放平台可以使用get和post
    def get(self, request):
        micro_url = 'https://api.weibo.com/oauth2/authorize?client_id={}&redirect_uri={}'.format(App_Key, MicroBlog_URL)

        return Response({'url': micro_url})

    # 使用新浪开放平台OAuth2/access_token接口, 只能使用post方法
    def post(self, request):
        # 获取code, 去访问token
        code = request.data.get('code', None)
        # 使用requests网络请求请求
        r = requests.post('https://api.weibo.com/oauth2/access_token', {
            'client_id': App_Key,
            'client_secret': App_Secret,
            'grant_type': 'authorization_code',
            'code': code,
            'redirect_uri': MicroBlog_URL
        })
        # 获取返回的对象
        print(r.json())
        access_token = r.json()['access_token']
        uid = r.json()['uid']
        if access_token:
            return Response({'msg': '登录成功', 'code': 200, 'uid': uid})
        return Response({'msg': '登录成功', 'code': 200})


# 使用微博登录时用到的回调地址
def micro_callback(request):
    code = request.GET.get('code', None)
    return redirect('http://127.0.0.1:8080/#/oauthCallback?code=' + code)
```

#### 前段部分
```js
<template>
  <div>
        <!-- 通过这个weiBoUrl地址可以进入微博的回调页，获取到code并传到后端 -->
      <a :href="weiBoUrl">微博</a>
  </div>
</template>
<script>
export default {
    data() {
        return {
            weiBoUrl: '',
        }
    },
    mounted() {
        // 获取微博授权页地址
      this.$axios.get('micro').then(res=>{
        this.weiBoUrl = res.data.url;
        console.log(res.data)
      }).catch(err=>{
        console.log(err)
      })
}
}
</script>
<style>
</style>
```