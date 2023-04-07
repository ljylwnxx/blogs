---
title: webpack升级
date: 2023-04-05 16:15:20
tags: webpack
categories: webpack
---
# 一、背景
由于项目越来越庞大复杂，打包时间也非常长，本地开发环境每次重启都要打包好久，正好借此契机对webpack做了一个升级。
# 二、升级过程
## 1. 先升级 webpack 和 webpack-cli
```scss
npm install --save-dev webpack@latest webpack-cli@latest  webpack-dev-server@latest webpack-merge@latest
```
webpack-merge升级以后，使用方式改为如下：
```js
const webpackMerge = require("webpack-merge");
```
```js
const { merge } = require('webpack-merge');
```
## 2. 执行npm start
在 package.json中scripts的start命令如下：
```json
"scripts": {
    "start": "cross-env NODE_ENV=dev webpack-dev-server --hot --progress --colors  --config ./webpack.dev.js",
}
```
### 1）--colors 报错
在v4版本中，我们可以使用 --colors或者 --color，但是在v5版本中只能使用 --color
调整命令：
```json
"scripts": {
    "start": "cross-env NODE_ENV=dev webpack-dev-server --hot --progress --color  --config ./webpack.dev.js",
}
```
### 2）devServer 中 disableHostCheck报错
```
devServer: {
    ... 
    disableHostCheck: true, 
    ... 
},
```
修改为：
```
devServer: {
    ... 
    allowedHosts: "all", 
    ... 
},
```
当设置为 'all' 时会跳过host检查。并不推荐这样做，因为不检查host的应用程序容易受到DNS重绑定攻击。
### 3）OpenBrowserPlugin 报错
打开浏览器的插件open-browser-webpack-plugin目前在 webpack5 中不能使用了，所以去掉。
+ webpack5 在开发环境可以通过 devServer.open 的方式去打开浏览器，但是不太建议，因为会导致构建速度明显变慢。
+ 可以利用 react-dev-utils 当中的 openBrowser 来实现，这个不会太影响构建速度，相当于自己写一个plugin。如下：
