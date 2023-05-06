---
title: webpack之require.context
date: 2023-05-06 11:55:23
tags: webpack
categories: webpack
---
# 了解webpack的require.context
## 关键代码
src/icons/index.js
```
const context = require.context("./svg", true, /\.svg$/)
context.keys().map(context)
```
main.js
```
import '@/icons'
```
webpack.base.config.js
```
{
    test: /\.svg$/,
    loader: "svg-sprite-loader",
    include: [resolve("src/icons")],
    options: {
        symbolId: "icon-[name]"
    }
},
{
    test: /\.(png|jpe?g|gif|svg)(\?.*)?$/,
    loader: "url-loader",
    exclude: [resolve("src/icons")],
    options: {
        limit: 10000,
        name: utils.assetsPath("img/[name].[hash:7].[ext]")
    }
},
```
## 为什么呢？
很多人跟我一样，一开始只想说，为什么这样就可以，why???
```
const context = require.context("./svg", true, /\.svg$/)
// 看看你是何方神圣
console.log(context)
context.keys().map(context)
```
看下面的图，理解一下
![](image.png)