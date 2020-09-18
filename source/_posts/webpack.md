---
title: webpack
date: 2020-09-17 14:12:58
tags:
- javascript
- webpack
- settings
categories:
- [web]
---

## webpack 常用loader & plugin

**说明：**
**plugin**相当于生命周期一样，在webpack运行到某个时刻的时候做一些事情。

### 汇总

```js
{
  "name": "webpack-test", // 项目名称
  "version": "1.0.0", // 当前版本
  "description": "", // 项目描述
  "main": "index.js", // 需要暴露出去的入口
  "private": true, // 是否私有 是否发布到npm上
  "scripts": { // npm指令
    ...
  },
  "author": "", // 开发者
  "license": "ISC" // 开源MIT
  "dependencies": {

  },
  "devDependencies": {
    "webpack": 使用webpack必备,
    "webpack-cli": 使用webpack命令,
    "file-loader": 打包图片(不推荐，使用更高级的loader),
    "url-loader": 打包图片(file-loader的升级版本),
    "css-loader": 打包css类文件,
    "style-loader": 打包css类文件，css-loader的下一步处理,
    "scss-loader": 处理scss类文件，处理完再经过css-loader处理,
    "autoprefixer": 处理css使之添加浏览器前缀,
    "html-webpack-plugin": 在打包结束后, 自动生成html文件,
    "clean-webpack-plugin": 在打包前。将dist清空,

  }
}
```

### [webpack](https://www.npmjs.com/package/webpack)

webpack is a **bundler** for modules **模块打包工具**。使之可支持ES Module，CommonJS，CMD，AMD。

### [webpack-cli]()

可以使我们在命令行中得意操作webpack指令。

### [file-loader](https://www.npmjs.com/package/file-loader)

#### 1.打包图片资源

```js
{
  ...
  module: {
    rules: [
      {
        test: /\.(png|jpe?g|gif)$/i,
        loader: 'file-loader',
        options: {
          // 设置打包图片的文件名
          name: '[name].[ext]', // name未打包之前的文件名 ext未打包文件的后缀名字
          // 打包图片到哪
          outputPath: 'imgs/'
        }
      }
    ]
  },
  ...
}
```

#### 2. 打包字体文件

```js
{
  ...
  module: {
    rules: [
      {
        test: /\.(eot|ttf|svg)$/,
        use: {
          loader: 'file-loader'
        }
      }
    ]
  }
}
```

### [url-loader](https://www.npmjs.com/package/url-loader)

将图片(较小的)转化为base64格式，打包进js，而较大的则打包成文件。是`file-loader`的**升级版**。

```js
{
  ...
  module: {
    rules: [
      {
        test: /\.(png|jpe?g|gif|svg)(\?.*)?$/,
        loader: 'url-loader',
        exclude: [resolve('src/components/icons')], // 针对某个范围下的图片进行处理
        options: {
          limit: 2048, // 小于2kb 则转化为base64
          name: '[name].[ext]'
        }
      },
    ]
  }
  ...
}
```

### [css-loader](https://www.npmjs.com/package/css-loader) & [style-lader](https://www.npmjs.com/package/style-loader)

css样式文件打包处理。

```js
{
  ...
  module: {
    rules: [
      {
        text: /\.css$/,
        use: ['style-loader', 'css-loader']
      }，
      {
        text: /\.scss$/,
        use: [
          'style-loader',
          {
            loader: 'css-loader',
            options: {
              importLoaders: 2, // 作用域@import引入的css文件
              modules: true // 启用cssModules模式
            }
          }，
          'scss-loader', // 特定的css类文件 就引入特定的loader包
          'postcss-loader' // 见下postcss-loader介绍
        ]
      }
    ]
  }
  ...
}
```

#### 1. importLoaders配置

假如一个scss文件中又引入另一个scss文件，如果不加配置，则只经过`css-loader`->`style-loader`处理，如果添加`importLoaders: 2`，表示通过**@import**引入的样式文件，至少通过之前的两个loader处理，即`postcss-loader`->`scss-loader`->`css-loader`->`style-loader`。

#### 2. modules

设置`modules: true`开启css模块化，避免产生css样式互相影响。

```js
import style from './index.css';

var img = new Image();
img.src = 'xxxxx';
img.classList.add(style);

document.getElementById('root').append(img);
```

### [postcss-loader](https://www.npmjs.com/package/postcss-loader)

打包css的样式时，进行预处理，可通过`postcss.config.js`配置。

使用`autoprefixer`，打包css3新特性时会带上浏览器兼容的前缀。
```
// postcss.config.js
module.exports = {
  plugins: [
    require('autoprefixer')
  ]
}
```

### [html-webpack-plugin](https://www.npmjs.com/package/html-webpack-plugin)

打包**结束**后，自动生成html，并自动将打包生成的js引入到这个html文件中。

```js
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
  plugins: [
    new HtmlWebpackPlugin(
      template: 'src/index.html'
    )
  ]
}
```

#### 1. template
`template`配置默认模版，将以这个html文件(`src/index.html`)为基础进行打包，`src/index.html`中的定制化结构都会打包到新的html中。

### [clean-webpack-plugin](https://www.npmjs.com/package/clean-webpack-plugin)

```js
const CleanWebpackPlugin = require('clean-webpack-plugin');

非官方的第三方插件，在webpack**打包前**将dist目录清空。

module.exports = {
  plugins: [
    new HtmlWebpackPlugin(...),
    new CleanWebpackPlugin()
  ]
}
```

-------

### 问题汇总

#### 1. 为什么项目下安装webpack，运行webpack报错找不到？

```js
webpack -v
command not found: webpack
```
**原因：**
因为默认从全局环境查找webpack, 因为我们并没有从全局安装webpack所以报错。如果是`package.json`中scripts指令执行，则不会出现这个问题，因为默认优先从工程中查找webpack包。

**解决：**
使用npx指令
```js
npx webpack -v
```

#### 2. 直接修改`webpack.config.js`文件名，不能正常打包

**原因：**
webpack指令默认查询的`webpack.config.js`，如果想执行正常操作需要通过`--config`修改指令。

**解决：**

```js
webpack --config 你修改后的文件路径
```

#### 3. webpack警告`The 'mode' option has not been set, ....`

**原因：**
在`webpack.config.js`中没有设置对应的mode

**解决：**
```js
modules.exports = {
  ...
  mode: production // production | development
  ...
}
```

#### 4. rules中多个loader使用use处理的机制

use的值是一个数组，秉承从下到上，从右往左执行的机制，比如 `use: ['style-loader', 'css-loader']`，一定是先经过`css-loader`处理，再经过`style-loader`处理。
