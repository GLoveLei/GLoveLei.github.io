---
title: 'Vue拦截器'
date: '2021/1/25 14:59:21'
categories:
   - Vue
toc_level: 3
version: 3
tags:
   - Vue拦截器
---
![cover](images/river.png)


# Vue拦截器

##### 一、路由守卫
在做项目时, 我们总是会判断一个用户有没有登录(例如: 当用户没有登录的时候，跳转到登录页面，已经登录的时候，不能跳转到登录页，除非后台token失效), 要实现这个功能的话, 就需要路由守卫来做.
```js
1 // 配置全局的vue路由拦截器, 导航守卫(路由守卫)
2 router.beforeEach((to,from,next) => {
3  console.log(to);  // 可以看出路由的去向
4  console.log(from);  // 可以看出从哪个路由跳转而来
5  const token = sessionStorage.getItem('token');  // 获取token
6  // 判断 是否是跳转到 某页, 如果是则必须登录才能跳转 
7  if (to.path in ['某页', '某页'] || token) {
8      next()  // 放行
9    } else {
10      alert('没有登陆, 不能操作');
11      next('/login?back=' + to.fullPath);  //跳转到登录页(带参数), path不能带参数, fullpath可以带参数
12     return  // 打断
13    }
14  });
```
##### 二、拦截器的作用及分类
拦截器的作用:

    一般来说，像数据交互之类的都要用到不同的身份验证，比如登录 token验证，验证用户
    
    是否登录，如果没有登录，该用户就不能操作登录之后的内容.
    
拦截器的分类:
     
    在请求或响应被 then 或 catch 处理前拦截他们，分为请求拦截器和响应拦截器.
    
##### 三、请求拦截器和响应拦截器
   一般在请求拦截器中增加标识token或其他请求配置，在响应拦截器中对统一错误或状态码进行处理(跳转统一页面如登录)
   
1、添加请求拦截器

请求拦截器获取token设置到axios请求头中，如果登录了, 那么自带token发送, 所有请求接口都具有这个功能
```js
1 // 添加请求拦截器
2 axios.interceptors.request.use(config=>{
3 // 在发送请求之前做些什么
4   return config;
5 }, error=> {
6    // 对请求错误做些什么
7    return Promise.reject8(error);
8   });
```   
2、添加响应拦截器

响应拦截器(在响应时会自动做的操作), 响应拦截器一般会对, 响应的错误信息进行处理。
```js
1 axios.interceptors.response.use(response => {
2  return response;  // 成功信息直接返回
}, error => {
3    // 错误信息需要处理
4    if (error.request.status === 401) {
5      window.location.href = '/';
6    }
7   return error
8 });
```
##### 四、封装get, post等方法
可根据封装好的axios，进行封装post,put,get,delete方法

必要参数:

1. url: 请求的url地址
2. params: 请求时携带的参数
3. headers: 请求时的头部

```js
1// get方法封装
2export function get(url, params, headers) {
3  return new Promise((resolve, reject) => {
4    axios.get(url, {params, headers}).then(res =>{
5      resolve(res)
6    }).catch(err => {
7      reject(err)
8    })
9  })
10 }
11
12 // post方法封装
13 export function post(url, params, headers) {
14  return new Promise((resolve, reject) => {
15    axios.post(url, params, headers).then(res =>{
16      resolve(res)
17    }).catch(err => {
18      reject(err)
19    })
20  })
21 }
22
23 // put方法封装
24 export function put(url, params, headers) {
25  return new Promise((resolve, reject) => {
26    axios.put(url, params, headers).then(res =>{
27      resolve(res)
28    }).catch(err => {
29      reject(err)
30    })
31  })
32 }
33
34 // delete方法封装
35 export function del(url, params, headers) {
36   return new Promise((resolve, reject) => {
37    axios.delete(url, {data: params, headers}).then(res => {
38       resolve(res)
39    }).catch(err => {
40      reject(err)
41    })
42  })
43 }
```

##### 五、使用封装方法
导入
```js
1 import { get, post, put, del } from '封装axios的js文件'
```
登录添加接口
```js
1 export const signIn = data => {
2  return post(
3    '/signIn',
4    data,
5   )
6 };
```
使用路径传参的接口
```js
1 export const book = (book_id) => {
2  return get(
3    'booksDetail/' + book_id + '/'
4   )
5 };
```
