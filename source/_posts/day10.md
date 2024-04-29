---
title: 'webshell'
date: '2021/2/3 14:59:21'
categories:
   - python
tags:
   - webshell
toc_level: 3
version: 3
---
![cover](images/river.png)


#### 一、什么是webshell
webshell是指在web服务器上的一个脚本程序，通过这个脚本程序可以上传下载文件、查看数据库信息以及调用一些系统命令等。Webshell 获取网站服务器的权限(shell)。
#### 二、webshell原理
本地可以通过远程连接的方式和网站的webshell建立连接，本地发送命令后，网站的shell脚本就会去执行此命令，从而达到管理网站的效果。
#### 三、webshell的分类有哪些？
    大马，小马，一句话木马等。
小马：

    体积小、功能少、一般他只有文件上传功能
大马:
    
    体积大、功能齐全、能够管理数据库、文件管理、对站点进行快速的信息收集，甚至能够提权。

一句话木马:

    短小精悍、功能强大、隐蔽性好、使用客户端可以快速管理webshell
    
#### 四、webshell的隐蔽性

有些恶意网页脚本可以嵌套在正常网页中运行，且不容易被查杀。

webshell可以穿越服务器防火墙，由于与被控制的服务器或远程主机交换的数据都是通过80端口传递的，因此不会被防火墙拦截。并且使用webshell一般不会在系统日志中留下记录，只会在网站的web日志中留下一些数据提交记录，没有经验的管理员是很难看出入侵痕迹的。

#### 五、后端部分
进行导包
```js
from dwebsocket import accept_websocket
import time
import paramiko
from threading import Thread
```
##### views.py配置
进行连接函数
```js
def make_ssh(host="127.0.0.1", username="root", password="123456"):
    """
    :host 主机地址
    :username 用户名，一般是root
    :password 密码
    :port ssh协议的端口,22
    """
    # 初始化一个ssh对象
    sh = paramiko.SSHClient()
    # 设置对象连接密钥规则
    sh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
    # 连接
    sh.connect(host, username=username, password=password, port=22)
    # 生成shell对象
    channle = sh.invoke_shell(term='xterm')
    return channle
```
接收函数
```js
def recv_ssh_msg(channle, ws):
    '''
        只管接收
    '''
    # 判断shell连接对象是否没有退出
    while not channle.exit_status_ready():
        # 接收过程可能会因为没有任何返回而报错
        try:
            buf = channle.recv(1024)  # 接收命令的执行结果
            ws.send(buf)  # 向Websocket通道返回
        # 接收不到会报错，但是报错没关系，继续重新尝试接受
        except:
            break
```
连接视图函数
```js
@accept_websocket
def webssh(request):
    if request.is_websocket:
        # 1. 获取到连接对象
        ws = request.websocket
        # 2. 初始化linux连接
        channel = make_ssh()
        # 3. 初始化linux数据接收线程，并开启
        recv_thread = Thread(target=recv_ssh_msg, args=(channel, ws))
        recv_thread.start()
        while 1:  # 主线程: 只管发送
            # 2. 阻塞ws接收发来的数据
            cmd = ws.wait()
            # cmd = ws.recv()
            if cmd:
                channel.send(cmd)  # 发送到linux 去执行
            else:  # 如果连接断开，那么cmd将会发一个空包
                break
        recv_thread.join()  # 回收子线程
        ws.close()  # 关闭ws连接
```

### 六、前端部分
在vue中导入插件
```js
import * as attach from 'xterm/lib/addons/attach/attach' // 安装插件适，可以使用attach去添加
import * as fit from 'xterm/lib/addons/fit/fit' // fit进行自适应大小的

import { Terminal } from 'xterm'
Terminal.applyAddon(attach) // 添加插件
Terminal.applyAddon(fit) // 添加插件
```

初始化黑窗口对象
```js
<template>
<div id="terminal">
    <!--黑色窗口-->
</div>
</template>


<script>
export default {
  data() {
    return {
    }
  },
mounted() {
        // 获取到了div标签
        let terminalContainer = document.getElementById('terminal')
        // 初始化黑窗口对象
        this.term = new Terminal(this.terminal)
        // 打开这个对象
        this.term.open(terminalContainer)

        new WebSocket('ws://127.0.0.1:8000/webssh/')
        this.terminalSocket = new WebSocket('ws://127.0.0.1:8000/webssh/')
        this.terminalSocket.onopen = function(){ // 连接成功触发该方法
            console.log('websocket is Connected...')
        }
        this.terminalSocket.onclose = function(){ // 连接关闭适触发的方法
            console.log('websocket is Closed...')
        }
        this.terminalSocket.onerror = function(){ // 连接出错触发的方法
            console.log('damn Websocket is broken!')
        }
        this.term.attach(this.terminalSocket)
    }
</script>
```