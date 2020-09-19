## [archer](https://github.com/fi3ework/hexo-theme-archer)

> 无法直接push到github仓库，固做一些备份处理

### 操作

#### 编辑文章结束后运行

```js
npx hexo g
npx hexo s
```
#### 如果期间失败清空再执行上面的操作

```js
npx hexo clean
```
#### 将主题下变更文件同步copy至相应的备注文件夹，再去git上传。

### 备份文件明细

|  备份存储地址  | 备份对象地址  |
|  ----  | ----  |
| /archer/layout  | /themes/archer/layout |
| /archer/_config.yml  | /themes/archer/_config.yml |
| /archer/avatar  | /themes/archer/source/avatar |
| /archer/intro  | /themes/archer/source/intro |

