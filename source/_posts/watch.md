---
title: watch
date: 2019-12-27 19:47:33
tags:
- Vue
categories:
- [web]
---
## watch 新旧值一致
> [Demo](https://codesandbox.io/s/watch-save-value-u1i6y)

### 场景
```javascript
<template>
  <div class="watch">
    <span>params.name:</span>
    <input type="text" v-model="params.name">
  </div>
</template>
```

```javascript
data() {
    return {
      params: {}
    };
  },
  watch: {
    params(newVal, oldVal) {
      console.log("普通方式oldVal", oldVal);
      console.log("普通方式newVal", newVal);
    }
  }

```
如上代码，input输入2，打印结果只打印一次且新旧值相等

![watch](/intro/watch_1.png)


### 官方的解释
> 注意：在变异(不是替换)对象或者数组时，旧值将与新值相等，因为它们的引用指向同一个对象/数组，Vue不会保留变异之前值的副本

### 解决方案

1. 监听到具体项

    ```javascript
    watch() {
        "params.name"(newVal, oldVal) {
        console.log("监听到具体项oldVal", oldVal);
        console.log("监听到具体项newVal", newVal);
        }
    }
    ```
    input输入2，再输入3，打印结果如下：
    ![watch](/intro/watch_2.png)


2. 监听一个计算属性
    ```javascript
    watch() {
        getParamsName(newVal, oldVal) {
        console.log("监听计算属性oldVal", oldVal);
        console.log("监听计算属性newVal", newVal);
        }
    },
    methods: {
        getParamsName() {
        return this.params.name;
        }
    }
    ```
    input输入2，再输入3，打印结果如下：
    ![watch](/intro/watch_3.png)

    **或者**你想直接监听一个计算属性，但是**返回整体**，而不是具体项
    ```javascript
    watch() {
        getParamsName(newVal, oldVal) {
        console.log("监听计算属性oldVal", oldVal);
        console.log("监听计算属性newVal", newVal);
        }
    },
    methods: {
        getParamsName() {
        return JSON.parse(JSON.stringify(this.params));
        }
    }
    ```
    input输入2，再输入3，打印结果如下：
    ![watch](/intro/watch_4.png)

<br/>
<br/>
<br/>
<br/>
