---
title: Git问题处理
date: 2021-06-30 12:00:08
tags:
- git
categories:
- [bugfix]
---

# Git 问题集中模版

## 修改文件大小写导致的问题

### 问题描述
如果修改同一个文件名，仅仅是英文大小写修改，那么git可能无法获悉这个修改。
### 解决途径
1. 预先设置大小写敏感
```js
// true 为忽略大小写
git config core.ignorecase false
```
2. 对已经造成大小写文件的问题
```js
// 将文件夹的名称从旧文件夹更改为新文件夹
git mv oldFile newFile

// 如果新文件夹已存在于您的存储库中，并且您想覆盖它并使用： -force
git mv -f oldFile newFile
```