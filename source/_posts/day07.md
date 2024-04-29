---
title: 'axios拦截器'
date: '2021/1/28 14:59:21'
categories:
   - vue
tags:
   - axios拦截器
toc_level: 3
version: 3
---
![cover](images/river.png)

## Axios拦截器使用教程

#### 一、安装axios
    npm install axios --save
    
#### 二、配置axios实例
在src文件下创建文件夹http，然后再里面创建index.js和api.js

index.js是用来分装axios和请求方法的，api.js是用来调用方法。

#### index.js
```js
// 配置axios基本的属性

const axios = require('axios')

axios.defaults.baseURL = 'http://127.0.0.1:8000' // 请求的接口主域名
axios.defaults.timeout = 5000 // 请求超时的设置 5s

// 请求拦截
axios.interceptors.request.use(
	//config 代表是你请求的一些信息
    config => {
        // 在请求发送之前的操作
        // 可以进行token的保持等......
        return config
    },
    error => {
        // 对错误请求的处理
        // 弹出错误请求消息
        console.log(error)
        return Promise.reject(error)
    }
)

//  response拦截器 响应拦截器 请求之后的操作
axios.interceptors.response.use(
    config => {
        return config
    },
    error => {
        return Promise.reject(error)
    }
)


/**
 * post方法，对应post请求
 * @param {String} url [请求的url地址]
 * @param {Object} params [请求时携带的参数]
 * @param {Object} headers [请求时的头部]
 **/

// get方法
export function get(url, params, headers) {
  return new Promise((resolve, reject) => {
    axios.get(url, {params, headers}).then(res => {
      resolve(res)
    }).catch(err => {
      reject(err)
    })
  })
}

//post方法
export function post(url, params, headers) {
  return new Promise((resolve, reject) => {
    axios.post(url, params, headers).then((res) => {
      resolve(res)
    }).catch((err) => {
      // debugger
      reject(err)
    })
  })
}

//put方法
export function put(url, params, headers) {
  return new Promise((resolve, reject) => {
    axios.put(url, params, headers).then((res) => {
      resolve(res)
    }).catch((err) => {
      // debugger
      reject(err)
    })
  })
}

//delete方法
export function del(url, params, headers) {
  return new Promise((resolve, reject) => {
    axios.delete(url, {data: params, headers}).then((res) => {
      resolve(res)
    }).catch((err) => {
      // debugger
      reject(err)
    })
  })
}

export default axios;

```

### 四、在api下进行使用所分装的方法
#### api.js
```js
import {get, post, put, del} from './index'

//get方法
export const 调用名 = parameter => {
  return get(
    '路由地址',
    parameter,
  )
}

// post方法
export const 调用名 = parameter => {
  return post(
    '路由地址',
    parameter,
  )
}

// put方法
export const 调用名 = parameter => {
  return put(
    '路由地址',
    parameter,
  )
}


// delete方法
export const 调用名 = parameter => {
  return del(
    '路由地址',
    parameter,
  )
}
```

### 五、最后在vue组件中进行使用

#### project.vue
```js
import {调用名} from "@/http/api.js";
export default {
  created() {
    调用名()
      .then(res => {
        console.log(res);
      })
      .catch(err => {
        console.log(err);
      });
  }
}
```

#### 六、总结
    在使用axios拦截器需要注意，先保证有axios的包，设置axios的请求接口然后
    进行分装请求拦截器，请求拦截器是在发送请求时所调用的方法，响应拦截器是
    在请求后进行的操作。然后分装get等方法进行使用，最后在vue组件中进行调用
    方法然后进行请求接口的访问。