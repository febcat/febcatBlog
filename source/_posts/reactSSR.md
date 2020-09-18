---
title: React同构
date: 2020-09-07 16:11:27
tags:
- React
- SSR
categories:
- [web]
- [server]
---

## 技术点汇总

**注意：**
> 如果用服务器端渲染，一定要让服务器端塞给 React 组件的数据和浏览器端一致。

**package.json**
```json
{
    ...
    "devDependencies": {
    "isomorphic-style-loader": "^5.1.0", // 服务端处理css的插件
    "nodemon": "^2.0.4", // 不用每次都重新node 监听文件改动自己重新打包
    "npm-run-all": "^4.1.5", // 可同时执行多个npm指令
    "babel-core": "^6.26.3",
    "babel-loader": "^7.1.5",
    "babel-preset-env": "^1.7.0",
    "babel-preset-react": "^6.24.1",
    "babel-preset-stage-0": "^6.24.1",
    "css-loader": "^3.6.0",
    "style-loader": "^1.2.1",
    "webpack": "4.16.0",
    "webpack-cli": "^3.3.12",
    "webpack-merge": "^5.1.3",
    "webpack-node-externals": "^2.5.2" // 服务端webpack配置项externals对应的包，防止将某些import的包(package)打包到bundl.js中, 如cdn引入的JQ之类
  },
  "dependencies": {
    "axios": "^0.20.0",
    "express": "^4.17.1",
    "express-http-proxy": "^1.6.2", // express代理中间件
    "react": "^16.13.1",
    "react-dom": "^16.13.1",
    "react-redux": "^7.2.1",
    "react-router-config": "^5.1.1", // 同构特殊的router模式对应的插件
    "react-router-dom": "^5.2.0",
    "redux": "^4.0.5",
    "redux-thunk": "^2.3.0"
  }
  ...
}
```

### 两端渲染

1. 服务端渲染

> 使用`renderToString(react element)` 返回一段HTML字符串

```js
// server/index.js
import { renderToString } from 'react-dom/server';
import express from 'express';

const app = express();

// 如果访问静态文件 就从public文件夹下去查询
app.use(express.static('public'));

export const render = () => {
    const content = renderToString(
        <div>
            ...
        </div>
    );

    return (`
        <html>
            <head>
                <title>
                    ssr
                </title>
            </head>
            <body>
                <div id="root">${content}</div>
                <script src='./index.js'></script> // 引入js 客户端第二次渲染
            </body>
        </html>
    `);
};

app.get('*', (req, res) => { // 执行渲染 将html返回给客服端, 客户端第一次渲染
    ...
    res.send(render())
}
```

2. 客户端渲染

> 使用`ReactDom.hydrate`来渲染。

`hydrate`由`ReactDOMServer`渲染的。 React将尝试将事件监听器附加到现有的标记。

相比较于`ReactDom.render`, 前者需要对整个应用做dom diff和dom patch, 花销较大，后者只会对text Content内容做patch。

使用`render`，一旦客户端渲染和服务端渲染dom比对有差异，客户端就会整个抛弃服务端渲染结构，重新渲染浏览器端产生的内容，也就造成了闪烁。

```js
// client/index.js
// html通过加载js 执行客户端代码 进行第二次渲染
const App = () => {
    return (
        <div>
            ...
        </div>
    )
};

ReactDom.hydrate(<App />, document.getElementById('root'));
```

### router路由配置

> 使用`react-router-config` `react-router-dom` 改用这种方式，是为了**服务端数据**的渲染

安装

```js
npm i react-router-config react-router-dom --save-dev
```
配置router json结构

```js
// Routes.js
import Home from './containers/Home';
import Login from './containers/Login';
import App from './App';

const routers = [
    {
        path: '/',
        component: App,
        key: 'app',
        routes: [
            {
                path: '/',
                component: Home,
                exact: true,
                key: 'home'
            }
        ]
    }
]

export default routers
```

1. 服务端

> 使用 `StaticRouter` `react-router-config`插件的`renderRoutes`方法
> server代码自此拆分为 server/index.js server/utils.js

```js
// server/utils.js
import React from 'react';
import { renderToString } from 'react-dom/server';
// 在服务端渲染中，实际上是不存在用户点击跳转的场景的，路径位置其实是一次请求->响应->渲染，始终不变的无状态静态模式
// StaticRouter 其实主要是方便开发者通过 context属性添加一些需要的属性
import { StaticRouter } from 'react-router-dom';
// renderRoutes 处理 routers的json配置
import { renderRoutes } from 'react-router-config';

// routes 就是 上面的json结构 从server/index.js 调用此方法时传入
export const render = (req, routes) => {
    const content = renderToString(
        // context 暂时为空对象，之后解释这里
        <StaticRouter location={req.path} context={{}}>
            <div>
                {renderRoutes(routes)}
            </div>
        </StaticRouter>
    )

    return (`
        <html>
            <head>
                <title>
                    ssr
                </title>
            </head>
            <body>
                <div id="root">${content}</div>
                <script src='./index.js'></script>
            </body>
        </html>
    `)
}
```

```js
// server/index.js
import express from 'express';
import { render } from './utils';
import routes from'../Routes';
import { matchRoutes } from 'react-router-config'；

const app = express();

app.get('*', (req, res) => {
    // 使用 react-router-dom 的 matchpath 只能获取到第一层的route 对于嵌套路由无法深层获取
    // const matchedRoutes = [];
    // routes.some(route => {
    //     const match = matchPath(req.path, route)

    //     if (match) {
    //         matchedRoutes.push(route)
    //     }
    // })

    const matchedRoutes = matchRoutes(routes, req.path);

    const html = render(req, routes);

    res.send(html)
});

const server = app.listen(3000);

```

**为何使用react-router-config的matchRoutes？**
因为`react-router-dom`的`matchpath`只能匹配到一层routes，对于嵌套路由处理是乏力的。

### redux配合router实现服务端数据渲染（脱水&注水）

安装

```js
npm i redux react-redux redux-thunk --save
```

router json中配置loadData

```js
// Routes.js
...
const routers = [
    {
        path: '/',
        component: App,
        key: 'app',
        routes: [
            {
                path: '/',
                component: Home,
                exact: true,
                key: 'home',
                loadData: Home.loadData // 加载数据的方法
            }
        ]
    }
]
...
```

业务组件Home代码添加loadData方法，redux方法不做详解，不是重点

```js
// Home.js 业务组件
import React, { Component } from 'react'
import { connect } from 'react-redux';
import { getHomeList } from './store/actions';

class Home extends Component {
    componentDidMount() {
        // 简单判定。如果有值则不再重新请求，这样避免首页服务端渲染后，第二次客户端渲染又一次请求接口，造成页面闪烁
        // 注意：这里需要考虑用户点击正常跳转的请求逻辑。故这里的请求并不能去除。
        if (!this.props.list.length) {
            this.props.getHomeList();
        }
    }

    getList() {
        const { list } = this.props;

        if (list && list.length) {
            return list.map(item => <div key={item.id}>{item.title}</div>);
        }

        return null
    }

    render() {
        return (
            <div className={style.home}>
                {this.getList()}
                <button onClick={() => {alert('click')}}>
                    click
                </button>
            </div>
        )
    }
}

const mapStateToProps = state => ({
    list: state.home.newsList
})

const mapDispatchToProps = dispatch => ({
    getHomeList() {
        dispatch(getHomeList())
    }
})

Home.loadData = store => {
   // 服务端渲染之前 将数据提前加载好
   return store.dispatch(getHomeList())
}

export default connect(mapStateToProps, mapDispatchToProps)(Home)
```

server逻辑作出调整，渲染前请求路由中的loadData方法，确保数据请求到在进行后面的渲染步骤

```js
// server/index.js
import express from 'express';
import { render } from './utils';
import routes from'../Routes';
import { matchRoutes } from 'react-router-config';
import { getServerStore } from '../stores';

const app = express();

app.get('*', (req, res) => {
    const store = getServerStore(); // redux store 这里不做解释
    const matchedRoutes = matchRoutes(routes, req.path);
    const promises = [];

    // 将需要执行的loadData收集 并push入prmises集合中
    matchedRoutes.forEach(item => {
        if (item.route.loadData) {
            const promise = new Promise((resolve, _) => {
                item.route.loadData(store).then(resolve).catch(resolve)
            })

            promises.push(promise)
        }
    });

    Promise.all(promises).then(_ => {
        const html = render(req, store, routes);

        res.send(html)
    })
});

const server = app.listen(3000);

```
在render之前，通过Promise来确保store中的数据获取，同时将获取的数据通过window.context={}形式传递给第二次的客户端渲染，这个过程就是注水。

```js
// server/utils.js
import React from 'react';
import { renderToString } from 'react-dom/server';
import { StaticRouter } from 'react-router-dom';
import { Provider } from 'react-redux';
import { renderRoutes } from 'react-router-config';

export const render = (req, store, routes) => {
    const content = renderToString(
        <Provider store={store}>
            <StaticRouter location={req.path} context={{}}>
                <div>
                    {renderRoutes(routes)}
                </div>
            </StaticRouter>
        </Provider>
    )

    return (`
        <html>
            <head>
                <title>
                    ssr
                </title>
            </head>
            <body>
                <div id="root">${content}</div>
                <script>
                    window.context = {
                        state: ${JSON.stringify(store.getState())}
                    }
                </script>
                <script src='./index.js'></script>
            </body>
        </html>
    `)
}
```

客户端获取store时，预读注水数据存储下来。

```js
// store/index.js
import { createStore, applyMiddleware, combineReducers } from 'redux';
import thunk from 'redux-thunk';
import { reducer as headReduce } from '../containers/components/Header/store';

const reducer = combineReducers({
    home: homeReduce
})

export const getServerStore = () => createStore(reducer, applyMiddleware(thunk)

export const getClientStore = () => {
    // 客户端 会默认获取全局window.ontext内容覆盖。
    // 同时配合业务组件Home下的componentDidMount中条件避免再次请求，导致页面闪烁
    const defaultState = window.context.state;

    return createStore(reducer, defaultState, applyMiddleware(thunk)
}

```

客户端渲染读取store中预先读取的注水数据，这个过程就是脱水。

```js
// client/index.js
import React from 'react';
import { hydrate } from 'react-dom';
import { BrowserRouter } from 'react-router-dom';
import routes from '../Routes';
import { getClientStore } from '../stores';
import { Provider } from 'react-redux';
import { renderRoutes } from 'react-router-config';

// 这里的store其实已经从window.context中获取了数据
const store = getClientStore();

const App = () => {
    return (
        <Provider store={store}>
            <BrowserRouter>
            <div>
                {renderRoutes(routes)}
            </div>
            </BrowserRouter>
        </Provider>
    )
}

hydrate(<App />, document.getElementById('root'));
```

**routes json的loadData方法**
因为在服务端中react的生命周期止步于componentWillMount, 在业务组件的componentDidMount中请求的数据自然在服务端无法获取，因此需要通过调用loadData方法来主动触发diaptch方法（使用redux）更新数据, 这样服务端就有数据了，同时第二次客户端渲染时候，通过一个条件判断，避免componentDidMount中重复请求数据。

**数据脱水和注水**
在服务端渲染的时候将获取到的数据赋值一个全局变量（注水），客户端创建的store以这个变量的值作为初始值（脱水）。

### 404页面配置

router json结构中配置相应页面

```js
// Routes.js
import Home from './containers/Home';
import NotFound from './containers/NotFound'
import App from './App';

const routers = [
    {
        path: '/',
        component: App,
        loadData: App.loadData,
        key: 'app',
        routes: [
            {
                path: '/',
                component: Home,
                exact: true,
                key: 'home',
                loadData: Home.loadData
            },
            {
                component: NotFound // 404页面
            }
        ]
    }
]

export default routers

```

404页面添加notFound属性

```js
import React from 'react'

class NotFound extends React.Component {
    componentWillMount() {
        // 使用StaticRouter可以给props附加 staticContext属性，依据这个属性就可以判定是否是服务端
        const { staticContext } = this.props;
        // 添加notFound属性
        staticContext && (staticContext.notFound = true);
    }

    render() {
        return (
            <p>Page Not Found 404!!!</p>
        )
    }
}

export default NotFound

```

将http状态修改为404

```js
// server/index.js
...
app.get('*', (req, res) => {
    ...
    Promise.all(promises).then(_ => {
        const context = {}
        const html = render(req, store, routes, context);
        // 当路由匹配到NotFound页面，会触发app.get('*')中间件，触发下面的判断
        // 如果有这个属性，说明router匹配的是NotFound页面
        if (context.notFound) {
            res.status(404)
        }

        res.send(html)
    })
    ...
}
...
```

### 配置redirect 301

`react-router-config`遇到`<Redirect />`会给context添加一些属性

```js
// 添加的结构如下
{
    action: 'REPLACE',
    loaction: {
        pathname: 'xx',
        search: '',
        hash: '',
        state: undefined
    },
    url: 'xx'
}
```

```js
// server/index.js
...
app.get('*', (req, res) => {
    ...
    Promise.all(promises).then(_ => {
        const context = {}
        const html = render(req, store, routes, context);

        // 服务端没有Dom Bom对象 自然无法执行<Redireact />的机制
        if (context.action === 'REPLACE') {
            res.redirect(301, context.url)

            return
        }

        if (context.notFound) {
            res.status(404)
        }

        res.send(html)
    })
    ...
}
...
```

2. 客户端

### css配置
> 服务端需要使用`isomorphic-style-loader`代替`css-loader`，因为服务端环境没有dom对象(window & document)

```js
npm i isomorphic-style-loader --save
```

## 可能遇到的问题汇总

### css-loader

1. 版本问题

```js
ValidationError: Invalid options object. CSS Loader has been initialized using an options object that does not match the API schema.
 - options has an unknown property 'localIdentName'. These properties are valid:
```

处理方法
> 安装3.6.0
```js
npm i css-loader@3.6.0 --save
```

2. 配置问题

```js
Error: Module build failed (from ./node_modules/css-loader/dist/cjs.js):
ValidationError: Invalid options object. CSS Loader has been initialized using an options object that does not match the API schema.
 - options has an unknown property 'localIdentName'. These properties are valid:
   object { url?, import?, modules?, sourceMap?, importLoaders?, localsConvention?, onlyLocals?, esModule? }
```

处理方法
> 查询文档，判断配置项是否正确， 特别指出服务端如果使用`isomorphic-style-loader`，是**不支持**localIdentName配置项的。
```js
// css-loader 变更
module: {
    rules: [
        {
            test: /\.css?$/,
            use: ['style-loader', {
                loader: 'css-loader',
                options: {
                    importLoaders: 1,
                    // modules: true,
                    // localIdentName: '[name]_[local]_[hash:base64:5]', // 旧的配置方式
                    modules: { // 新的配置方式 localIdentName在modules下
                        localIdentName: '[name]_[local]_[hash:base64:5]'
                    },

                }
            }]
        }
    ]
}
```

### 静态文件

1. 服务单报错找不到种种静态文件

看是否在服务端有如下配置

```js
app.use(express.static('public'));
```
