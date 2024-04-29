---
title: 'OpenAI ChatGPT接入微信 wechatbot 搭建微信聊天机器.'
date: '2023/07/10 14:59:21'
categories:
   - openai
tags:
   - 人工智能
toc_level: 3
version: 3
---

![cover](images/wechat.png)

[](about:blank#OpenAI-ChatGPT%E6%8E%A5%E5%85%A5%E5%BE%AE%E4%BF%A1-wechatbot-%E6%90%AD%E5%BB%BA%E5%BE%AE%E4%BF%A1%E8%81%8A%E5%A4%A9%E6%9C%BA%E5%99%A8%E4%BA%BA "OpenAI ChatGPT接入微信 wechatbot 搭建微信聊天机器人")OpenAI ChatGPT接入微信 wechatbot 搭建微信聊天机器人
=============================================================================================================================================================================================================================================

[](about:blank#%E4%B8%80%E3%80%81%E5%87%86%E5%A4%87%E5%B7%A5%E4%BD%9C "一、准备工作")一、准备工作
-------------------------------------------------------------------------------------

### [](about:blank#%E9%A6%96%E5%85%88%E6%98%AF%E6%B3%A8%E5%86%8CChatGPT "首先是注册ChatGPT")首先是注册ChatGPT

*   1、获取自己 ChatGPT 的 apikey [获取地址](https://platform.openai.com/account/api-keys) 登入后即可获取
    
*   2、一台VPS境外服务器（本次演示系统 Debian 10 64）[Vultr 购买地址，点此进入](https://www.vultr.com/?ref=8941832-8H)：按时计费，最低6$/月。
    
*   3、一个微信号（建议使用小号测试）
    
*   4、下载并安装FinalShell SSH工具
    

Windows版下载地址:[点此下载](http://www.hostbuf.com/downloads/finalshell_install.exe)

macOS版下载地址: [点此下载](http://www.hostbuf.com/downloads/finalshell_install.pkg)

[](about:blank#%E4%BA%8C%E3%80%81ChatGPT%E6%8E%A5%E5%85%A5%E5%BE%AE%E4%BF%A1 "二、ChatGPT接入微信")二、ChatGPT接入微信
----------------------------------------------------------------------------------------------------------

### [](about:blank#%E5%BC%80%E5%A7%8B%E9%85%8D%E7%BD%AE%E6%9C%8D%E5%8A%A1%E5%99%A8 "开始配置服务器")开始配置服务器

安装 Node 环境

```
sudo apt update && sudo apt upgrade

curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -

sudo apt-get install nodejs
```

安装`git`命令

```
apt install git
```

克隆 Wechat-Chatgpt 项目[点此进入项目地址](https://github.com/fuergaosi233/wechat-chatgpt)

```
git clone https://github.com/fuergaosi233/wechat-chatgpt.git
```

进入Wechat-Chatgpt项目

```
cd wechat-chatgpt
```

### [](about:blank#%E5%AE%89%E8%A3%85%E4%BE%9D%E8%B5%96%E5%B9%B6%E5%88%9B%E5%BB%BA%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6 "安装依赖并创建配置文件")安装依赖并创建配置文件

切换到new-wechatgpt分支

```
git checkout new-wechatgpt
```

安装依赖

```
npm install
```

创建配置文件

```
cp config.yaml.example config.yaml

cp .env.example .env
```

修改配置文件：API = 就是你在openAI官网上生成的apikey

```
vi .env
```

.env内容如下：

```
CHAT_GPT_EMAIL=
CHAT_GPT_PASSWORD=
CHAT_GPT_RETRY_TIMES=
CHAT_PRIVATE_TRIGGER_KEYWORD=
OPENAI_PROXY=
NOPECHA_KEY=
CAPTCHA_TOKEN=
OPENAI_API_KEY=填写你的API_KEY
```

XShell保存退出指令：首先按ESC进入Command模式，然后输入“：wq”，回车就可以保存并退出了

修改模型

打开文件：

```
cd wechat-chatgpt


vi node_modules/chatgpt/build/index.js
```

找到73行，修改模型为：CHATGPT\_MODEL=”text-davinci-003”

```
text-davinci-003
```

### [](about:blank#%E4%BF%9D%E5%AD%98%E5%A5%BD%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%E5%90%8E%EF%BC%8C%E5%90%AF%E5%8A%A8%E6%89%A7%E8%A1%8C%E4%B8%8B%E6%96%B9%E7%9A%84%E5%90%AF%E5%8A%A8%E5%91%BD%E4%BB%A4%EF%BC%8C%E5%B0%B1%E5%8F%AF%E4%BB%A5%E5%92%8C%E6%9C%BA%E5%99%A8%E4%BA%BA%E8%81%8A%E5%A4%A9%E4%BA%86%EF%BC%81 "保存好配置文件后，启动执行下方的启动命令，就可以和机器人聊天了！")保存好配置文件后，启动执行下方的启动命令，就可以和机器人聊天了！

```
npm run dev
```

### [](about:blank#%E5%A6%82%E6%9E%9C%E9%9C%80%E8%A6%81%E5%9C%A8%E5%90%8E%E5%8F%B0%E5%AE%88%E6%8A%A4%E8%BF%9B%E7%A8%8B%E8%BF%90%E8%A1%8C%EF%BC%8C%E9%82%A3%E4%B9%88%E5%8F%AA%E9%9C%80%E8%BF%90%E8%A1%8C%E4%B8%8B%E9%9D%A2%E5%91%BD%E4%BB%A4 "如果需要在后台守护进程运行，那么只需运行下面命令")如果需要在后台守护进程运行，那么只需运行下面命令

```
nohup command

nohup chatgpt &amp;</dev/null   &amp; 
```

这样即使你断开VPS，机器人也会在后台运行。

运行测试结果：  
![cover](images/wechat-02.png)

### [](about:blank#%E6%80%BB%E7%BB%93 "总结")总结

现在chatgpt已经越来越发达，任何一种接入方式都会让生活变的方便,如果普及以后都不用打开chatgtp官网链接vpn进行查询，方便高效实现高效ai工具  
唯一美中不足的就是微信有防止接入机制容易封号,愿以后可以更加方便

  

