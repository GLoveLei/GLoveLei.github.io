---
title: 'Vue路由钩子'
date: '2021/1/26 14:59:21'
categories:
   - Vue
tags:
   - Vue路由钩子函数
toc_level: 3
version: 3
---
![cover](images/river.png)

## 路由钩子函数
#### 一、路由钩子函数
路由钩子作用就是拦截导航栏，就是在跳转的时候进行判断

    vue-router 提供的导航钩子主要用来拦截导航，让它完成跳转或取消
    
##### 路由钩子分为三类： 全局的、单个路由独享的、或者组件级


#### 二、全局钩子

主要包括beforeEach和aftrEach,这类钩子主要作用于全局,一般用来判断权限,以及以及页面丢失时候需要执行的操作。

```js
1 beforeEach函数有三个参数：
2 - to:router即将进入的路由对象
3 - from:当前导航即将离开的路由
4 - next:Function,进行管道中的一个钩子，如果执行完了，则导航的状态就是 confirmed （确认的）；否则为false，终止导航。
5 可以使用router.beforeEach注册一个全局的 before 钩子
6 router.beforeEach((to, from, next) => { // ... })
7
8 after 钩子没有 next 方法，不能改变导航
9 router.afterEach(route => { // ...})
```
#### 三、单个路由钩子
主要用于写某个指定路由跳转时需要执行的逻辑

```js
1 {
2        path: '',
3        component: '',
4        meta: '',
5        beforeEnter: (to, from, next) => {     
6            //进入时执行代码块
7        },
8        beforeLeave: (to, from, next) => {
9            //离开时执行代码块
10        }
11    }
```

#### 四、组件内的钩子
主要包括 beforeRouteEnter和beforeRouteUpdate ,beforeRouteLeave,这几个钩子都是写在组件里面也可以传三个参数(to,from,next)。

```js
1 <script>
2  Vue.component('LifeCircle',{
3    template: '#life-circle',
4    data () {
5      return {
6      }
7    },
8    // 初始化阶段钩子函数
9    beforeCreate () { //表示组件创建前的准备工作（ 初始化事件和生命周期 ）     
10      //组件未创建，数据拿不到
11      console.log('1');
12    },
13    created () { // 组件创建结束
14      console.log('2')
15    },
16    beforeMount () { 
17      console.log( '3' )
18    },
19    mounted () {     //表示组件装载结束
20      console.log('4')
21    }
22 </script>
```
