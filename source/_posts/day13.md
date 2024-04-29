---
title: 'docker操作 容器、镜像 dockerfile'
date: '2021/2/6 14:59:21'
categories:
   - docker
tags:
   - docker操作 容器、镜像 dockerfile
toc_level: 3
version: 3
---
![cover](images/lake.png)

#### 一、介绍
虚拟化操作:把一个操作系统分成多个操作系统

vmware:过于庞大，太过于慢，但是确实安装出来的虚拟机：**隔离性很好**
docker：更快，更轻巧，但是隔离性不好

#### 二、docker安装-CentOS7
安装必要的⼀些系统⼯具
```js
yum install -y yum-utils device-mapper-persistent-data lvm2
```
添加软件源信息
```js
yum-config-manager --add-repo https://mirrors.aliyun.com/dockerce/linux/centos/docker-ce.repo
```
更新并安装Docker-CE
```js
yum makecache fast yum -y install docker-ce
```
开启Docker服务
```js
service docker start
```

#### 二、下载镜像
下载镜像
```js
docker pull alpine
```
查看所有镜像
```js
docker images
```
启动容器：docker run
```js
docker run -d -i -t 镜像id /bin/bash
```
查看所有运行着的容器：docker ps
先进入容器进行配置，配置有关webssh连接的事项
```js
docker attach 镜像id
```
安装ssh
```js
yum install openssh-server
```
生成当前主机的ssh-key
```js
ssh-keygen -t dsa -f /etc/ssh/ssh_host_dsa_key
ssh-keygen -t rsa -f /etc/ssh/ssh_host_rsa_key
ssh-keygen -t ecdsa -f /etc/ssh/ssh_host_ecdsa_key -N
ssh-keygen -t dsa -f /etc/ssh/ssh_host_ed25519_key
```

在容器里启动ssh服务，让外界可以连接
```js
/usr/sbin/sshd
```

#### 三、Dockerfile
镜像的定制实际上就是定制每一层所添加的配置、文件。如果我们可以把每一层修改、安装、构建、操作的命令都写入一个脚本，用这个脚本来构建、定制镜像，那么无法重复的问题、镜像构建透明性的问题、体积的问题就都会解决。这个脚本就是 Dockerfile。


在一个空白目录中，建立一个文本文件，并命名为 Dockerfile ：
```js
 mkdir mynginx
 cd mynginx
 touch Dockerfile
```
构建dockerfile
```js
docker build -t nginx:v3
...
```
docker build 命令进行镜像构建
```js
docker build [选项] /home/tools/dockerfile
```
