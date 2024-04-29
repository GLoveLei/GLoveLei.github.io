---
title: '直播'
date: '2021/2/5 14:59:21'
categories:
   - python
tags:
   - 直播
toc_level: 3
version: 3
---
![cover](images/river.png)

#### 一、直播简介
 直播分为：现场直播和观看直播，现场直播称为推流，观看直播称为拉流，一套流程走下去就可完成直播

#### 二、搭建HTTP-Flv协议
##### 在云主键进行部署环境
创建保存的目录
```js
mkdir /usr/local/nginx-flv
# 创建了一个安装目录
mkdir /home/flv-tools
# 存储所需软件
```
下载所需文件
```js
wget https://nginx.org/download/nginx-1.18.0.tar.gz
```
下载直播模块
```js
git clone https://github.com/winshining/nginx-http-flv-module
```
解压nginx和直播模块
```js
tar -zxvf nginx-1.18.0.tar.gz nginx-1.18.0/
unzip nginx-http-flv-module.zip
```
进入nginx目录下进行配置
```js
cd nginx-1.18.0

# --prefix: 配置安装路径
#--add-module: 添加安装插件
./configure --prefix=/usr/local/nginx-flv --add-module=/home/flv-tools/nginx-http-flv-module-master
```
编译及安装
```js
make 
make install 
# 或者使用
make && make install 
```

#### 三、配置文件修改
```js
vim /usr/local/nginx-flv/conf/nginx.conf


worker_processes  1;

rtmp_auto_push on;
rtmp_auto_push_reconnect 1s;
rtmp_socket_dir /tmp;

rtmp{
	out_queue 4096;
	out_cork 8;
	max_streams 128;
	timeout 15s;
	drop_idle_publisher 15s;
	log_interval 5s;
	log_size 1m;
	server {
		listen 1935; # 推流端口
		server_name zege;
		
		application live {	# 配置推流地址
			live on; # 打开推流
			# gop_cache on;
			# rtmp://123.123.123.123:1935/live/test
		}
	}
}

events {
    worker_connections  1024;
}
```
配置文件http拉流部分
```js
http {
	include       mime.types;
	default_type  application/octet-stream;
	sendfile        on;
	keepalive_timeout  65;
	server {
		listen       8080; # 拉流通过8080去拉流
		# http://123.123.123.123:8080/live/?port=1935&stream=test
		server_name  localhost;
		location /live {
			flv_live on;
			chunked_transfer_encoding  on;
			add_header 'Access-Control-Allow-Origin' '*';
			add_header 'Access-Control-Allow-Credentials' 'true';
		}
	}
}
```
·关闭旧有服务
```js
/usr/local/nginx-rtmp/sbin/nginx -s stop
/usr/local/nginx-rtmp/sbin/nginx -s reload
# 重启
```
开启新服务
```js
/usr/local/nginx-flv/sbin/nginx -c /usr/local/nginx-flv/conf/nginx.conf
```

#### 四、打开OBS进行推流配置
推流地址为:rtmp://39.106.221.128:22/live/123

#### 五、打开VLC进行拉流
拉流地址为:http://39.106.221.128:22/live?port=1935&app=live&stream=123


#### 六、Vue实现拉流
安装**flv.js**开源工具，进行拉流
```js
cnpm install flv.js --save
```
导包
```js
import flv from 'flv.js'
```
构建页面标签，播放标签
```js
<video id="videoElement" controls muted>
  	Your browser is too old which doesn't support HTML5 video.
</video>
```
进行初始化
```js
 mounted() {
    var videoElement = document.getElementById('videoElement');
    var flvPlayer = flv.createPlayer({
      type: 'flv',
      enableWorker: true,     //浏览器端开启flv.js的worker,多进程运行flv.js
      isLive: true,           //直播模式
      hasAudio: false,        //关闭音频
      hasVideo: true,
      // cors: true,
      stashInitialSize: 128,
      enableStashBuffer: false, //播放flv时，设置是否启用播放缓存，只在直播起作用。
      // url: 'http://192.168.2.234/flv/323223618780001'
      // url: 'http://39.105.79.238:8080/live?port=1935&app=live&stream=test'
      url: 'http://47.93.48.154:8080/live?port=1935&app=live&stream=test'
    })
    flvPlayer.attachMediaElement(videoElement);
    flvPlayer.load();
    flvPlayer.play();
}
```