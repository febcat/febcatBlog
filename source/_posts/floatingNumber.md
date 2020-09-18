---
title: 浮点数计算
date: 2019-11-01 20:33:00
tags:
- javascript
categories:
- [web]
header_image: /intro/post-bg.jpg
---

## toPrecision vs toFixed

> [参考](https://github.com/camsong/blog/issues/9)

> 在计算的中间过程不要使用, 只用于最终结果。

数据处理时，这两个函数很容易混淆。它们的共同点是把数字转成字符串供展示使用。

### 不同点
* toPrecision 是处理精度，精度是从左至右第一个不为0的数开始数起。
* toFixed 是小数点后指定位数取整，从小数点开始数起。

### 相同点
&nbsp;&nbsp;&nbsp;&nbsp;两者都能把数字转成字符串供展示使用，同时对多余数字做凑整处理。

### 两者的bug

+ toFixed
  有的人用 toFixed 来做四舍五入，但一定要知道它是有 Bug 的。
  如：1.005.toFixed(2) 返回的是 1.00 而不是 1.01。

  **原因**： 1.005 实际对应的数字是 1.00499999999999989，在四舍五入时全部被舍去！

  **解法**：使用专业的四舍五入函数 Math.round() 来处理。但 Math.round(1.005 * 100) / 100 还是不行，因为 1.005 * 100 = 100.49999999999999。还需要把乘法和除法精度误差都解决后再使用 Math.round。

- toPrecision
  在保留15位的时候就有偏差，但是一般不会保留15位
  ```javascript
  (12.000000000999999).toPrecision() // 12.000000000999998
  ```

<br/>
<br/>

---------

<br/>
<br/>

## 开发中解决方案

### 数据展示

  1 保留位数
  ```javascript
  // 12能大部分解决0001和0009的问题。12 = 整数 + 小数
  parseFloat(1.4000000000000001.toPrecision(12)) === 1.4  // True
  ```

  2 舍取不必要的零
  ```javascript
  (12.1200).toPrecision() // 12.12
  ```
### 数据运算类

  对于运算类操作，如 +-*/，就不能使用 toPrecision 了。正确的做法是把小数转成整数后再运算。
  解决精确计算可以使用bugnumber.js


### 其他方法

[toLocaleString()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number/toLocaleString) 里面可以配置options 但是尴尬的是最低支持ie11 而且只能四舍五入算法

<br/>
<br/>
<br/>
<br/>
