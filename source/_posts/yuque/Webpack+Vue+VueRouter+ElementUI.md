
---

title: Webpack+Vue+VueRouter+ElementUI

date: 2019-07-03 19:49:33 +0800

categories: 前端

tags: [教程]

---


<a name="3qSdE"></a>
# 本文重点

- 搭建Webpack开发环境
- 引入Vue
- 引入Vue-Router
- 引入ElementUI

<a name="bPzMF"></a>
# 搭建Webpack开发环境
<a name="YSZLE"></a>
### 1. 搭建npm

```shell
npm init -y
# 打印以下信息
{
  "name": "lesson02",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}
```

<a name="RSFLK"></a>
### 2. 集成webpack
<a name="fJRBR"></a>
#### 2.1 安装webpack webpack-cli
```shell
yarn add -D webpack webpack-cli
```

<a name="jGOc9"></a>
#### 2.2 创建webpack.config.js
```javascript
module.exports = {
    entry: './src/index.js', // 指明webpack入口文件
    mode: 'development' // 指明webpack打包模式，development、production
}
```

<a name="qVBVg"></a>
#### 2.3 配置package.json
```json
{
  "name": "lesson02",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "build": "webpack --config webpack.config.js" // 添加build脚本，webpack并配置webpack.config.js
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "webpack": "^4.35.0",
    "webpack-cli": "^3.3.4"
  }
}
```

<a name="JvMF8"></a>
#### 2.4 创建入口文件并打包
创建 `src/index.js` 文件，在终端执行 `npm run build` 
```shell
$ webpack --config webpack.config.js
Hash: 5b90c629ada6fed12c59
Version: webpack 4.35.0
Time: 42ms
Built at: 2019-06-23 10:44:26
  Asset      Size  Chunks             Chunk Names
main.js  3.77 KiB    main  [emitted]  main
Entrypoint main = main.js
[./src/index.js] 0 bytes {main} [built]
```

执行完命令后，webpack会自动创建 `dist/main.js` ，此时webpack已经帮我们将源码打包进了 `main.js` 。

<a name="6mySL"></a>
### 3. Webpack打包html文件
webpack打包html文件需要集成 `html-webpack-plugin` 插件
<a name="22qUQ"></a>
#### 3.1 创建index.html
在 `src` 目录下，创建index.html文件
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Webpack+Vue+Element Demo</title>
</head>
<body>
    <h1>Welcome to Webpack!</h1>
</body>
</html>
```

<a name="wFWIT"></a>
#### 3.2 配置HtmlWebpackPlugin
在 `webpack.config.js` 文件里配置 `html-webpack-plugin` 插件
```javascript
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
    entry: './src/index.js',
    mode: 'development',
    plugins: [
        new HtmlWebpackPlugin({
            template: './src/index.html'
        })
    ]
}
```

<a name="Hsnrm"></a>
#### 3.3 webpack打包
执行 `npm run build` 使用webpack打包。
```shell
$ webpack --config webpack.config.js
Hash: ceb7a1f973b58b7b2266
Version: webpack 4.35.0
Time: 267ms
Built at: 2019-06-23 11:04:24
     Asset       Size  Chunks             Chunk Names
index.html  361 bytes          [emitted]
   main.js   3.77 KiB    main  [emitted]  main
Entrypoint main = main.js
[./src/index.js] 0 bytes {main} [built]
Child html-webpack-plugin for "index.html":
     1 asset
    Entrypoint undefined = index.html
    [./node_modules/html-webpack-plugin/lib/loader.js!./src/index.html] 512 bytes {0} [built]
    [./node_modules/webpack/buildin/global.js] (webpack)/buildin/global.js 472 bytes {0} [built]
    [./node_modules/webpack/buildin/module.js] (webpack)/buildin/module.js 497 bytes {0} [built]
        + 1 hidden module
✨  Done in 0.81s.
```
执行完成后，在 `dist` 目录下同时生成 `index.html` 和 `main.js` 文件，其中 `index.html` 文件自动引入了 `main.js` 
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Webpack+Vue+Element Demo</title>
</head>
<body>
    <h1>Welcome to Webpack!</h1>
<script type="text/javascript" src="main.js"></script></body>
</html>
```

<a name="LgZB0"></a>
### 4. 使用webpack-dev-server实现热更新
<a name="WjklP"></a>
#### 4.1 安装 `webpack-dev-server` 
执行 `npm install -S webpack-dev-server` 
<a name="Hbc5F"></a>
#### 4.2 配置 `package.json` 
```json
{
  "name": "lesson02",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "build": "webpack --config webpack.config.js",
    "start": "webpack-dev-server" // 配置webpack-dev-server
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "html-webpack-plugin": "^3.2.0",
    "webpack": "^4.35.0",
    "webpack-cli": "^3.3.4",
    "webpack-dev-server": "^3.7.2"
  }
}
```
<a name="z6Ur8"></a>
#### 4.3 启动webpack-dev-server
```shell
$ webpack-dev-server
ℹ ｢wds｣: Project is running at http://localhost:8080/
ℹ ｢wds｣: webpack output is served from /
ℹ ｢wds｣: Content not from webpack is served from /Users/luzhiyong/workspace/Nugget/FrontEnd/Demos/element-ui/lesson02
ℹ ｢wdm｣: Hash: 621a07b0efc37e879bd7
Version: webpack 4.35.0
Time: 412ms
Built at: 2019-06-23 18:32:41
     Asset       Size  Chunks             Chunk Names
index.html  361 bytes          [emitted]
   main.js    359 KiB    main  [emitted]  main
Entrypoint main = main.js
[0] multi (webpack)-dev-server/client?http://localhost ./src/index.js 40 bytes {main} [built]
[./node_modules/ansi-html/index.js] 4.16 KiB {main} [built]
[./node_modules/ansi-regex/index.js] 135 bytes {main} [built]
[./node_modules/html-entities/index.js] 231 bytes {main} [built]
[./node_modules/loglevel/lib/loglevel.js] 7.68 KiB {main} [built]
[./node_modules/strip-ansi/index.js] 161 bytes {main} [built]
[./node_modules/webpack-dev-server/client/index.js?http://localhost] (webpack)-dev-server/client?http://localhost 4.29 KiB {main} [built]
[./node_modules/webpack-dev-server/client/overlay.js] (webpack)-dev-server/client/overlay.js 3.51 KiB {main} [built]
[./node_modules/webpack-dev-server/client/socket.js] (webpack)-dev-server/client/socket.js 1.53 KiB {main} [built]
[./node_modules/webpack-dev-server/client/utils/createSocketUrl.js] (webpack)-dev-server/client/utils/createSocketUrl.js 2.77 KiB {main} [built]
[./node_modules/webpack-dev-server/client/utils/log.js] (webpack)-dev-server/client/utils/log.js 964 bytes {main} [built]
[./node_modules/webpack-dev-server/client/utils/reloadApp.js] (webpack)-dev-server/client/utils/reloadApp.js 1.63 KiB {main} [built]
[./node_modules/webpack-dev-server/client/utils/sendMessage.js] (webpack)-dev-server/client/utils/sendMessage.js 402 bytes {main} [built]
[./node_modules/webpack/hot sync ^\.\/log$] (webpack)/hot sync nonrecursive ^\.\/log$ 170 bytes {main} [built]
[./src/index.js] 0 bytes {main} [built]
    + 18 hidden modules
Child html-webpack-plugin for "index.html":
     1 asset
    Entrypoint undefined = index.html
    [./node_modules/html-webpack-plugin/lib/loader.js!./src/index.html] 512 bytes {0} [built]
    [./node_modules/lodash/lodash.js] 527 KiB {0} [built]
    [./node_modules/webpack/buildin/global.js] (webpack)/buildin/global.js 472 bytes {0} [built]
    [./node_modules/webpack/buildin/module.js] (webpack)/buildin/module.js 497 bytes {0} [built]
ℹ ｢wdm｣: Compiled successfully.
```
浏览器访问[http://localhost:8080/](http://localhost:8080/)即可看到index.html的内容

<a name="xMyMi"></a>
### 5. Webpack打包css文件
<a name="FwHWT"></a>
#### 5.1 安装 css-loader、style-loader
```shell
yarn add -D css-loader style-loader
```
<a name="iJQKA"></a>
#### 5.2 配置css-loader、style-loader
在 `webpack.config.js` 中配置加载 `css-loader` 和 `style-loader` 
```javascript
const HtmlWebpackPlugin = require('html-webpack-plugin')

module.exports = {
    entry: './src/index.js',
    mode: 'development',
    module: {
        rules: [
            {
                test: /\.css$/,
                use: [
                    'style-loader',
                    'css-loader'
                ]
            },
        ]
    },
    plugins: [
        new HtmlWebpackPlugin({
            template: './src/index.html'
        })
    ]
}
```
<a name="QPPjR"></a>
### 6. Webpack打包less文件
<a name="G3rAI"></a>
#### 6.1 安装less-loader
需要提前安装好 `css-loader` 和 `style-loader` 
```shell
yarn add -D less-loader less
```
<a name="6O217"></a>
#### 6.2 配置less-loader
```javascript
const HtmlWebpackPlugin = require('html-webpack-plugin')

module.exports = {
    entry: './src/index.js',
    mode: 'development',
    module: {
        rules: [
            {
                test: /\.css$/,
                use: [
                    'style-loader',
                    'css-loader'
                ]
            },
            {
                test: /\.less$/,
                use: [
                    'style-loader',
                    'css-loader',
                    'less-loader'
                ]
            },
        ]
    },
    plugins: [
        new HtmlWebpackPlugin({
            template: './src/index.html'
        })
    ]
}
```

<a name="Dhi3B"></a>
# 引入Vue
<a name="Kbr6T"></a>
### 1. 安装vue、vue-loader、vue-template-compiler
`vue-loader` 和 `vue-template-compiler` 这两个插件是webpack打包vue代码所必须的，关于这两个插件的更多信息可以参考[起步 | Vue Loader](https://vue-loader.vuejs.org/zh/guide/#手动设置)。
<a name="rD8LY"></a>
### 2. 配置webpack.config.js支持加载vue
```javascript
const HtmlWebpackPlugin = require('html-webpack-plugin')
const VueLoaderPlugin = require('vue-loader/lib/plugin')

module.exports = {
    entry: './src/index.js',
    mode: 'development',
    module: {
        rules: [
            {
                test: /\.vue$/, // 通过规则匹配将vue-loader应用到所有扩展名为vue的文件
                loader: 'vue-loader'
            }
        ]
    },
    plugins: [
        new VueLoaderPlugin(), // 这个插件必须引用，它的职责是将定义过的其他规则应用到vue文件的相应语言块
        new HtmlWebpackPlugin({
            template: './src/index.html'
        })
    ]
}
```

<a name="ZU6QK"></a>
### 3. 引用vue
<a name="MWWge"></a>
#### 3.1 修改index.html
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Webpack+Vue+Element Demo</title>
</head>
<body>
   <div id="app">
       {{ message }}
   </div>
</body>
</html>
```
 
<a name="8Q83N"></a>
#### 3.2 修改index.js
```javascript
import Vue from 'vue';

new Vue({
    data: {
        message: 'Hello Vue!'
    }
}).$mount("#app")
```

<a name="Psnit"></a>
#### 3.3 配置vue文件路径
完成上述配置后，运行项目会遇到以下错误信息：
> [Vue warn] : You are using the runtime-only build of Vue where the template option is not available. Either pre-compile the templates into render functions, or use the compiler-included build. (found in root instance)

原因是因为在vue的package.json中引入的是vue.common.js，vue打包时会生成三个文件，一个是 runtime only 的文件 vue.common.js，一个是 compiler only 的文件 compiler.js，一个是 runtime + compiler 的文件 vue.js。如果要使用 template 这个属性的话就一定要用 compiler.js，那么，引入 vue.js 是最恰当的。<br />配置webpack.config.js, 将vue指向 `vue/dist/vue.js` 。
```javascript
const HtmlWebpackPlugin = require('html-webpack-plugin')
const VueLoaderPlugin = require('vue-loader/lib/plugin')

module.exports = {
    entry: './src/index.js',
    mode: 'development',
    resolve: {
        alias: {
            'vue': 'vue/dist/vue.js'
        }
    },
    module: {
        rules: [
            {
                test: /\.vue$/,
                loader: 'vue-loader'
            }
        ]
    },
    plugins: [
        new VueLoaderPlugin(),
        new HtmlWebpackPlugin({
            template: './src/index.html'
        })
    ]
}
```

<a name="U7sPU"></a>
### 4. 引入App.vue文件
<a name="qRtjD"></a>
#### 4.1 创建App.vue文件
```html
<template>
    <div>
        <header>header</header>
    </div>
</template>

<script>
export default {
    
}
</script>

<style>
    header {
        background: red;
    }
</style>
```
<a name="xC0eR"></a>
#### 4.2 引入App.vue文件
修改 `index.js` 文件
```javascript
import Vue from 'vue';

import App from './App.vue';

new Vue({
    el: '#app',
    render: h => h(App)
})
```

<a name="SkKfn"></a>
# 引入Vue-Router
<a name="e5pgC"></a>
### 1. 安装vue-router
```shell
yarn add vue-router
```
<a name="XwPCz"></a>
### 2. 引入vue-router
<a name="Eg3ks"></a>
#### 2.1 在 `src/views` 目录下创建 `Home.vue` 和 `About.vue` 

```html
<template>
    <div>
        Home Page
    </div>
</template>
```

```html
<template>
    <div>
        About Page
    </div>
</template>
```

<a name="WzLjS"></a>
#### 2.2 在 `index.js` 中引入 `vue-router` 
```javascript
import Vue from 'vue';
import VueRouter from 'vue-router';

import App from './App.vue';
import Home from './views/Home.vue';
import About from './views/About.vue';

Vue.use(VueRouter)

const routes = [
    { path: '/home', component: Home },
    { path: '/about', component: About }
]

const router = new VueRouter({
    routes
})

new Vue({
    router,
    el: '#app',
    render: h => h(App)
})
```

<a name="lBopn"></a>
#### 2.3 配置 `router-link`  `router-view` 
在 `App.vue` 里配置 `router-link` 与 `router-view` 
```html
<template>
    <div>
        <header>
            <router-link to="home">主页</router-link>
            <router-link to="about">关于我们</router-link>
        </header>
        <router-view></router-view>
    </div>
</template>

<script>
export default {
    
}
</script>

<style>
    header {
        height: 60px;
        background: #ff9744;
    }
</style>
```

<a name="HLde3"></a>
# 引入Element UI
<a name="z1iMl"></a>
### 1. 安装 `element-ui` 、 `file-loader` 
分别安装 `element-ui` 与 `file-loader` ,安装 `file-loader` 的原因是 `element-ui` 中有一些字体文件需要在webpack打包时加载。
```shell
yarn add element-ui
yarn add file-loader
```

<a name="pF1W7"></a>
### 2. 配置 `file-loader` 
在 `webpack.config.js` 中配置加载 `file-loader` 

```javascript
const HtmlWebpackPlugin = require('html-webpack-plugin')
const VueLoaderPlugin = require('vue-loader/lib/plugin')

module.exports = {
    entry: './src/index.js',
    mode: 'development',
    resolve: {
        alias: {
            'vue': 'vue/dist/vue.js'
        }
    },
    module: {
        rules: [
            {
                test: /\.vue$/,
                loader: 'vue-loader'
            },
            {
                test: /\.css$/,
                use: [
                    'style-loader',
                    'css-loader'
                ]
            },
            {
                test: /\.less$/,
                use: [
                    'style-loader',
                    'css-loader',
                    'less-loader'
                ]
            },
            {
                test: /\.(eot|svg|ttf|woff|woff2)$/,
                loader: 'file-loader'
            }
        ]
    },
    plugins: [
        new VueLoaderPlugin(),
        new HtmlWebpackPlugin({
            template: './src/index.html'
        })
    ]
}
```

<a name="T1jf6"></a>
### 3. 引入 `element-ui` 
在 `index.js` 中引入 `element-ui` 、 `theme-chalk/index.css` ，详细的配置可以参考[Element-UI快速上手]()

```javascript
import Vue from 'vue';
import VueRouter from 'vue-router';

import ElementUI from 'element-ui'; 
import 'element-ui/lib/theme-chalk/index.css';

import App from './App.vue';
import Home from './views/Home.vue';
import About from './views/About.vue';

Vue.use(VueRouter)
Vue.use(ElementUI);

const routes = [
    { path: '/home', component: Home },
    { path: '/about', component: About }
]

const router = new VueRouter({
    routes
})

new Vue({
    router,
    el: '#app',
    render: h => h(App)
})
```

<a name="XMTpZ"></a>
### 4. 使用element-ui
修改 `App.vue` 使用 `element-ui` 进行布局
```html
<template>
    <div>
        <el-container>
            <el-aside>Aside</el-aside>
            <el-container>
                <el-header>Header</el-header>
                <el-main>Main</el-main>
            </el-container>
        </el-container>
    </div>
</template>

<script>
export default {
    
}
</script>

<style>
    header {
        height: 60px;
        background: #ff9744;
    }
</style>
```


