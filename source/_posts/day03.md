---
title: '子父组件'
date: '2021/1/27 14:59:21'
categories:
   - Vue
tags:
   - Vue父子组件 子父组件
toc_level: 3
version: 3
---
![cover](images/river.png)

## 组件
#### 什么是组件？

   组件是Vue中的一个重要概念，是一个可以重复使用的Vue是可以复用的Vue实例，它拥有独一无二的组件名称，它可以扩展HTML元素，以组件名称的方式作为自定义的HTML标签。因为组件是可复用的Vue实例，所以它们与new Vue（）接收相同的选项，例如data，computed、watch、methods以及生命周期钩子等。仅有的例外是像el这样根实例特有的选项。

## 父子组件通信
 #### 父组件
 ·父组件在组件上定义了一个自定义事件childFn，事件名为parentFn用于接受子组件传过来的message值。
 ```js
<!-- 父组件 -->
<template>
    <div class="test">
      <test-com @childFn="parentFn"></test-com>
      <br/> 
      子组件传来的值 : {{message}}
    </div>
</template>

<script>
export default {
    // ...
    data() {
        return {
             message: ''
        }
    },
    methods: {
       parentFn(payload) {
        this.message = payload;
      }
    }
}
</script>
```
#### 子组件
·子组件是一个buttton按钮，并为其添加了一个click事件，当点击的时候使用$emit()触发事件，把message传给父组件。
```js
<!-- 子组件 -->
<template> 
<div class="testCom">
    <input type="text" v-model="message" />
    <button @click="click">Send</button>
</div>
</template>
<script>
export default {
    // ...
    data() {
        return {
          // 默认
          message: '我是来自子组件的消息'
        }
    },
    methods: {
      click() {
            this.$emit('childFn', this.message);
        }
    }    
}
</script>
```
#### 小结

·通过"props down , events up"我们就简单的实现了父子组件之间的双向传值

## 子父组件通信
第一种方法是直接在子组件中通过this.$parent.event来调用父组件的方法

#### 父组件
```js
<template>
  <div>
    <check></check>
  </div>
</template>
<script>
  import check from '@/components/check';
  export default {
    components: {
      check
    },
    methods: {
      father() {
        console.log('传值成功');
      }
    }
  };
</script>
```
#### 子组件
```js
<template>
  <div>
    <button @click="check()">点击</button>
  </div>
</template>
<script>
  export default {
    methods: {
      check() {
        this.$parent.father();
      }
    }
  };
</script>
```
·第二种方法是在子组件里用$emit向父组件触发一个事件，父组件监听这个事件就行了。

#### 父组件
```js
<template>
  <div>
    <check @fatherMethod="father"></check>
  </div>
</template>
<script>
  import check from '@/components/check';
  export default {
    components: {
      check
    },
    methods: {
      father() {
        console.log('传值成功');
      }
    }
  };
</script>
```
#### 子组件
```js
<template>
  <div>
    <button @click="check()">点击</button>
  </div>
</template>
<script>
  export default {
    methods: {
      check() {
        this.$emit('father');
      }
    }
  };
</script>
```

·第三种是父组件把方法传入子组件中，在子组件里直接调用这个方法
#### 父组件
```js

<template>
  <div>
    <check :father="father"></check>
  </div>
</template>
<script>
  import check from '@/components/check';
  export default {
    components: {
      check
    },
    methods: {
      father() {
        console.log('传值成功');
      }
    }
  };
</script>
```
#### 子组件
```js

<template>
  <div>
    <button @click="check()">点击</button>
  </div>
</template>
<script>
  export default {
    props: {
      father: {
        type: Function,
        default: null
      }
    },
    methods: {
      check() {
        if (this.father) {
          this.father();
        }
      }
    }
  };
</script>
```