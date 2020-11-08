---
layout: post
title: '不使用 vue-cli 与 Vue 模版，使用 Vue2.x + webpack4.x 从零开始一步步搭建项目框架'
categories: 前端
tags: webpack Vue
excerpt: '本文介绍了在不使用 vue-cli 与 Vue 模版的情况下，如何使用 Vue2.x + webpack4.x 从零开始一步步搭建项目框架。'
---

## 说明

这是我根据慕课网上的一个课程 [Vue+Webpack打造todo应用](https://www.imooc.com/learn/935) 过程一步步搭下来的框架，去掉了业务相关的逻辑。  
项目最终的效果包括了引入Vue框架；使用CSS预处理器；使用babel；引用图片等静态资源；区分开发环境与生成环境，并做相应优化等。基本接近真正做项目时候的配置。  
**但是！！**毕竟是我个人根据练习课程搭的框架，跟真实工作可能有区别，**请谨慎直接用于工作环境！！！**

项目的最终成果看这里：https://gitee.com/Dandelion_/vue-webpack-scaffold  
**Tips**：项目里面的 `commit` 对应文章的每一小节，所以大家善用 [`git checkout <commit>`](https://github.com/geeeeeeeeek/git-recipes/wiki/2.5-%E6%A3%80%E5%87%BA%E4%B9%8B%E5%89%8D%E7%9A%84%E6%8F%90%E4%BA%A4) 命令可以很方便地切换到某个 `commit` 看当前版本的文件变化哦。当然直接看项目的 [`commit` 列表](https://gitee.com/Dandelion_/vue-webpack-scaffold/commits/master)也行啦。

---

## 0. 前言

首先不得不说 `vue-cli` + `vue-webpack` 模版真的很方便，`vue init webpack my-vue-project` 就搭好框架了，而且开发环境生产环境都有了。`npm run dev` 启动开发环境，`npm run build` 发布生产环境。几个命令全部搞定了。但是模版有时候可能不够灵活，或者我们想修改其中一些东西，面对这些需求的时候，理解怎么搭建起这个 `vue` + `webpack` 的环境还是很有用的。那最快捷的方式，当然是自己从头开始搭一遍啦。心动不如行动，走~~

---

## 1. 新建项目

因为不用 vue-cli 和 vue 模版，所以一开始我们就建个空文件夹。

```bash
mkdir my-vue-project
cd my-vue-project
npm init # 初始化项目。这时候会问你一些问题，比如项目名称、作者之类的，照着提示回答问题就好
```

## 2. 安装基本的 npm 包

首先肯定要安装 `vue`  和 `webpack`。然后 `webpack` 4.x已经把 cli 单独拎出来了，所以还要安装 `webpack-cli`；然后因为 `webpack` 本身其实直接能处理的只有 js 资源，是通过各种 loader 让其他资源可以被 `webpack` 打包处理的。那么现在我们要用 `vue` 写单文件组件（就是 `.vue` 文件），所以就还要安装 `vue-loader`。

```bash
npm install vue vue-loader webpack webpack-cli --save-dev # --save-dev表示这些只是开发环境所需的依赖
```

## 3. 添加项目各种入口文件

入口文件都是很普遍的那种，比如根目录下的 `index.html`，`src` 文件夹下的 `main.js`，`app.vue` 等等，我就不一一解释了。

添加入口文件后的项目目录如下：
```
.
├── dist/
|	├── ...
├── src/
|	├── main.js
|   ├── app.vue
├── node_modules/
|	├── ...
├── index.html
├── package.json
```
更详细的项目结构和具体文件内容可以看 [这个commit](https://gitee.com/Dandelion_/vue-webpack-scaffold/tree/d2a8cde3f4b668c75b5f92f4990fd5acbaf70bbb/)

## 4. 添加 webpack 配置文件

在项目根目录下添加 `webpack.config.js` 文件。内容如下：
```javascript
const path = require('path');
const VueLoaderPlugin = require('vue-loader/lib/plugin');

module.exports = {
    entry: path.join(__dirname, 'src/main.js'), // 项目总入口js文件
    // 输出文件
    output: {
        path: path.join(__dirname, 'dist'), // 所有的文件都输出到dist/目录下
        filename: 'bundle.js'
    },
    module: {
        rules: [{
                // 使用vue-loader解析.vue文件
                test: /\.vue$/,
                loader: 'vue-loader'
            },
            {
                test: /\.css$/,
                // 要加上style-loader才能正确解析.vue文件里的<style>标签内容
                use: ['style-loader', 'css-loader']
            }
        ]
    },
    plugins: [
        new VueLoaderPlugin() // 最新版的vue-loader需要配置插件
    ]
};
```

## 5. 添加构建脚本

在 `package.json` 文件的 `scripts` 属性里添加 `build` 脚本

```json
   "scripts": {
     "test": "echo \"Error: no test specified\" && exit 1",
+    "build": "webpack --config webpack.config.js"
   },
```

因为我们添加了一些引用，比如 `style-loader`、`css-loader` 等，所以也要安装相应的包。
```bash
npm i style-loader css-loader vue-template-compiler --save-dev
```

安装好依赖后运行 `npm run build` 来构建项目。
构建成功后在项目根目录启动一个静态资源服务器，然后浏览就可以看到以下页面了。

![](/assets/img/posts/2018-10/1.jpg)

## 6. 添加图片、CSS 预处理器等 loader

图片 loader 用的是 `url-loader`，它依赖于 `file-loader`，比 `file-loader` 多了一个可以比小于一定大小的图片直接转化成 base64 的形式插入到 html 页面，可以减少网络请求。

`CSS 预处理器` 我选的是 `SASS`。

安装依赖：
```bash
npm i file-loader url-loader node-sass sass-loader --save-dev
```

修改 `webpack.config.js` 
```javascript
		module.exports = 
			{
                 test: /\.css$/,
                 // 要加上style-loader才能正确解析.vue文件里的<style>标签内容
                 use: ['style-loader', 'css-loader']
+            },
+            {
+                test: /\.scss$/,
+                use: [
+                    // 处理顺序是从sass-loader到style-loader
+                    'style-loader',
+                    'css-loader',
+                    'sass-loader'
+                ]
+            },
+            {
+                test: /\.(gif|jpg|jpeg|png|svg)$/i,
+                use: [{
+                    loader: 'url-loader',
+                    options: {
+                        // 当文件大小小于limit byte时会把图片转换为base64编码的dataurl，否则返回普通的图片
+                        limit: 8192,
+                        name: 'dist/assest/images/[name]-[hash:5].[ext]' // 图片文件名称加上内容哈希
+                    }
+                }]
             }
         ]
     },
     ...
```

测试一下：

添加 `src/assets/styles/global.scss` 文件
```scss
+.play {
+    background-image: url('../images/play.png');
+    background-repeat: no-repeat;
+    width: 64px;
+    height: 64px;
+    padding: 0;
+    border: 0;
+    background-color: transparent;
+    margin: 0;
+    cursor: pointer;
+}
```

在 `src/main.js` 引用  `global.scss`
```javascript
 import Vue from 'vue'; // 从node_modules引入vue类库
 import App from './app.vue'; // ES6 语法，相当于 import { default as App } from './app.vue'。因为app.vue用过的是export default {...}，所以可以这样写
  
+import './assets/styles/global.scss';
+
 new Vue({
     el: '#app',
     components: {
```

运行 `npm run build`，刷新页面，能看到如下效果：  
![](/assets/img/posts/2018-10/2.jpg)
button 的背景图是 1.97kb，小于 8192byte，可以看到图片已经被转换成 base64 的内容了。

## 7. 添加 `postcss-loader` + `autoprefixer`，自动添加 `css` 浏览器前缀

安装依赖：
```bash
npm i postcss-loader autoprefixer --save-dev
```

新增 `postcss` 配置文件 `postcss.config.js`
```javascript
const autoprefixer = require('autoprefixer');

module.exports = {
    plugins: [
        autoprefixer({
            browsers: ['last 5 versions']
        })
    ]
};
```

修改 `webpack.config.js`
```javascript
	...
	module: {
        rules: [
			...
			{
                test: /\.css$/,
                use: [
                    // 要加上style-loader才能正确解析.vue文件里的<style>标签内容
                    'style-loader',
                    {
                        loader: 'css-loader',
                        options: {
                            importLoaders: 1
                        }
                    },
                    'postcss-loader'
                ]
            },
            {
                test: /\.scss$/,
                use: [
                    // 处理顺序是从sass-loader到style-loader
                    'style-loader',
                    'css-loader',
                    {
                        loader: 'postcss-loader',
                        options: {
                            sourceMap: true
                        }
                    },
                    'sass-loader'
                ]
            },
            ...
```

测试一下：

修改 `index.html`
```html
     <h1>一个用 vue + webpack 搭建的脚手架框架</h1>
     <div id="app"></div>
     <button class="play"></button>
+    <div class="flex">
+        <div class="flex-item"></div>
+        <div class="flex-item"></div>
+        <div class="flex-item"></div>
+    </div>
     <script src="/dist/bundle.js"></script>
 </body>
```

修改 `global.scss`
```scss

+.flex {
+    display: flex;
+    border: solid #000 2px;
+    width: 300px;
+    height: 100px;
+    .flex-item {
+        flex: 1;
+        background-color: #1296db;
+        border: solid #fff 1px;
+    }
 }
```

运行 `npm run build`，刷新页面，能看到如下效果：  
![](/assets/img/posts/2018-10/3.jpg)
可以看到 `flex` 已经加上了浏览器前缀了。

## 8. 添加 `babel-loader`，转译 `es6` 代码为 `es5` 代码

添加.babelrc文件
```json
{
    "presets": [
        "env"
    ],
    "plugins": [
        "transform-vue-jsx"
    ]
}
```

安装依赖
```bash
npm i babel-loader@7 babel-core babel-preset-env --save-dev
```

修改 `webpack.config.js`，`module.rules` 再加一条：
```javascript
{
    test: /\.js$/,
    exclude: /(node_modules|bower_components)/, // 不处理这两个文件夹里的内容
    loader: 'babel-loader'
}
```

测试一下：

修改 `main.js`，加上以下代码
```javascript
// 测试babel-loader
class Person {
    constructor(name, age) {
        this.name = name;
        this.age = age;
    }

    sayHello() {
        console.log(`Hello, my name is ${this.name}`);
    }
}
```

在浏览器查看输出，如下图：
![](/assets/img/posts/2018-10/4.jpg)

如果注释掉 `webpack.config.js` 里 `babel-loader` 这条规则，则编译之后如下：
![](/assets/img/posts/2018-10/5.jpg)
可以看到 `babel-loader` 已经起作用了。

## 9. 添加 `html-webpack-plugin`，自动生成 `index.html` 的内容

添加 `HtmlWebpackPlugin`，在 `dist` 文件夹也生成 `index.html` 页面，而且会自动加上对项目入口 `js` 文件的引用，也就是说我们自己写的 `index.html` 可以不用再手动指定 `js` 文件了，它会把 `webpack` 配置里的 `entry` 当中指定的 `js` 都插入到生成的 `index.html` 中，而且当输出的文件添加上 `hash` 值时也可以自动跟踪。

安装依赖
```bash
npm i html-webpack-plugin --save-dev
```

修改 `webpack.config.js`
```javascript
 const path = require('path');
 const VueLoaderPlugin = require('vue-loader/lib/plugin');
+const HtmlWebpackPlugin = require('html-webpack-plugin');
  
 module.exports = {
     entry: path.join(__dirname, 'src/main.js'), // 项目总入口js文件
     // 输出文件
     output: {
         path: path.join(__dirname, 'dist'),
-        filename: 'bundle.js'
+        filename: 'bundle-[hash].js' // 输出文件的名称加上hash值
     },
     module: {
         rules: [{
             ...
         ]
     },
     plugins: [
-        new VueLoaderPlugin() // 最新版的vue-loader需要配置插件
+        new VueLoaderPlugin(), // 最新版的vue-loader需要配置插件
+        new HtmlWebpackPlugin({
+            filename: 'index.html', // 生成的文件名称
+            template: 'index.html', // 指定用index.html做模版
+            inject: 'body' // 指定插入的<script>标签在body底部
+        })
     ]
 };
```

修改 `index.html`，把原来引用 `bundle.js` 的 `<script>` 去掉。

然后我们 `build` 一下，看 `dist` 文件夹的输出。
![](/assets/img/posts/2018-10/6.jpg)
可以看到 `dist` 文件夹下多了 `index.html` 文件，而且文件内容自动加上了编译出来的 `bundle.js` 文件的引用。

## 10. 添加 `clean-webpack-plugin`，每次 `build` 之前可以自动先清除输出文件夹

如果我们对输出的文件名称加上了哈希值的话，每次修改之后的哈希值都不一样，就是每次都生成了一个新的文件，那旧的那些其实就是多余的文件了（上面那张图的 `dist` 文件夹可以看出来）。因此我们可以用 `clean-webpack-plugin` 这个插件，在每次 `build` 之前先清除输出文件夹。

安装依赖
```bash
npm i clean-webpack-plugin --save-dev
```

修改 `webpack.config.js`，`plugins` 数组添加一项
```javascript
 const path = require('path');
 const VueLoaderPlugin = require('vue-loader/lib/plugin');
 const HtmlWebpackPlugin = require('html-webpack-plugin');
+const CleanWebpackPlugin = require('clean-webpack-plugin');
  
 module.exports = {
     ...
     plugins: [
        new VueLoaderPlugin(), // 最新版的vue-loader需要配置插件
        new HtmlWebpackPlugin({
            filename: 'index.html', // 生成的文件名称
            template: 'index.html', // 指定用index.html做模版
            inject: 'body' // 指定插入的<script>标签在body底部
        }),
+        new CleanWebpackPlugin(['dist'])
    ]
 };
```

## 11. 添加 `webpack-dev-server`，配置更友好的开发环境

`webpack-dev-server` 可以在本地启动一个服务器，而且当有任何文件修改的时候会自动重新打包，并且刷新浏览器页面。此外 devServer 还有很多其他配置项，让我们可以更方便的开发。

安装依赖
```bash
npm i webpack-dev-server cross-env --save-dev
```
用上 `cross-env` 是因为我们接下来要修改 `package.json` 里的 `scripts`，而不同的平台写`scripts` 的方式不一样。

修改 `package.json` 里的 `scripts`
```
    ...
     "scripts": {
         "test": "echo \"Error: no test specified\" && exit 1",
-        "build": "webpack --config webpack.config.js"
+        "build": "cross-env NODE_ENV=production webpack --config webpack.config.js",
+        "dev": "cross-env NODE_ENV=development webpack-dev-server --config webpack.config.js"
     },
   ....
```

修改 `webpack.config.js`，这次改得比较多，可以看这里的 [commit detail](https://gitee.com/Dandelion_/vue-webpack-scaffold/commit/da0471178e6a0da3ca598158baacec4015f6f71d)

配置好以后我们可以运行 `npm run dev` 命令启动一个服务器了

## 12. 配置生产环境 css 单独分离打包，方便浏览器缓存

安装依赖
```bash
npm i mini-css-extract-plugin --save-dev
```

修改 `webpack.config.js`，改动的地方看这个 [commit detail](https://gitee.com/Dandelion_/vue-webpack-scaffold/commit/85d341fe0590bfd10cc14e123c36762be33670e3)

## 13. 单独打包类库文件

因为类库文件是不用像业务代码一样经常更新的，单独打包可以让它们在浏览器里缓存，提高加载速度。

修改 `webpack.config.js`，改动的地方看这个 [commit detail](https://gitee.com/Dandelion_/vue-webpack-scaffold/commit/fd5bb510394e532d19a9b277c0c9ddf60527d9c3)

---

到这里基本的配置都完成了，一个基础的 `vue` 项目框架就搭好啦，接下来我们只需要专注于添加自己项目的业务逻辑页面和代码就好了。愉快地敲(xie)代(bug)码去吧~~

**PS：**这个是目前为止用到的配置，其实真实生成环境要做的优化应该还有不少，比如图片压缩处理等等，大家要根据自己的项目需要来扩展哦。如果以后有用到其他的插件比较通用的应该会更新这篇文章。嗯，也就是不定期无规律看心情更新2333【逃。。】
