---
title: 正则表达式的兼容问题
date: 2021-06-23 14:52:19
tags:
- javascript
categories:
- [bugfix]
---

# RegExp Lookbehind 反向定言 兼容
> [ES9 RegExp Lookbehind Assertions](https://github.com/tc39/proposal-regexp-lookbehind)

## 问题描述
修改公共方法以用支持省略小数位多余的零，这里使用的是正则。上线后反馈用户使用safari浏览器打不开页面。

**正则方法如下**
```js
import NP from 'number-precision';

function formatPercent(num, type: FormatPercentType = 'multiply', decimal = 2) {
    if (num === 0 || num === '0') {
        return '0%';
    }

    if (!!(num * 1)) {
        return (type === 'multiply'
            ? `${NP.times(num, 100).toFixed(decimal)}%`
            : type === 'divide'
            ? `${NP.divide(num, 100).toFixed(decimal)}%`
            : `${num.toFixed(decimal)}%`
        ).replace(/(?<=\.\d*)(0+)(?=%)|(\.0+)(?=%)/, '');
    }

    return EMPTY;
}

```

## 问题分析

### chrome浏览器运行
运行是正常的
![$attrs](./bugfix_regexp_chrome.png)

### safari浏览器运行
发现报错
![$attrs](./bugfix_regexp_safari.png)

这是因为**?<=**和**?<!** 这类反向定言正则属于ES9阶段。目前兼容性还不是很好，babel也不能处理这个情况。

![$attrs](./bugfix_can_i_use.png)
