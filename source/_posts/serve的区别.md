---
title: 浅析vue项目中npm run dev和npm run serve的区别
date: 2023-03-13 11:03:47
tags: vue
categories: 知识点
---
# 先做一个提问，你们运行项目的时候使用什么命令来启动项目呢？
通常我们在开发前端项目的过程中，经常需要使用npm run dev或者npm run serve命令来启动项目，同样都是启动项目的命令，但是有些时候运行npm run serve会报错，而运行npm run dev则正常运行，那这两者到底有什么区别呢？
## 在理解这两者的区别之前，我们应该要理解这两条命令是什么意思。
# 首先看看npm是什么？
npm实际上是nodejs官方提供的包管理平台，npm提供了一个命令行工具npm-cli，在我们使用npm这个命令时，我们实际是通过node运行一个名为npm-cli.js的脚本
# 在看看npm install命令
在构建项目时，通过npm install命令，会在项目目录下生成一个名为node_modules的文件夹
主要用于存放包管理工具，下载安装的包的文件夹，也就是项目依赖的外部模块的缓存，比如element-ui，echarts等一些组件库通过npm i 安装之后会被下载复制到node_modules文件夹下
# 然后看看运行npm run ×××命令的原理
大家都知道，通过npm run serve 和npm run dev启动项目，可以通过npm run build命令打包项目，那么在运行npm run 时，到底发生了什么呢？
在项目目录下，我们可以看到一个名为package.json的文件，该文件是对项目、模块包的描述，在package.json文件中，有一个scripts的字段
![](npm区别.png)
***
我们可以看到，运行npm run serve命令启动项目的项目中，scripts中有一个serve字段，运行npm run dev 命令中存在一个dev字段，那么这时候我们可以大概了解到，在运行npm run命令时，实际上是去package.json这个文件的scripts中寻找对应的×××，然后去执行对应的命令，那么是不是我们想要运行npm run dev命令的话，只需要把scripts中的serve改成dev就能通过npm run dev来启动项目呢？
答案是可以，确实将serve改成dev之后，我们运行npm run dev命令可以启动项目

# 既然可以通过更改字段来进行转换，那为什么会有npm run serve 和npm run dev这两种模式呢？
正如上面我们所说的，运行npm run serve的时候，实际是运行vue-cli-service serve这条命令，在上面介绍中，我们可以看到scripts中，不管serve还是dev对应的命令都是vue-cli-service serve，那么dev和serve到底区别在哪里呢？
我们会发现，原来npm run dev和npm run serve本质上是运行vue-cli不同版本下启动项目的脚本，dev在vue-cli2.0中使用，serve在vue-cli3.0中才开始使用。








