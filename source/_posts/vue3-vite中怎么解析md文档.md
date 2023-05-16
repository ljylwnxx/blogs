---
title: vue3+vite中怎么解析md文档
date: 2023-05-15 22:34:31
tags: markdown
categories: vue3
---
# 「摘要」
Vue的markdown解析库有很多，如markdown-it、vue-markdown-loader、marked、vue-markdown等，这些库都大同小异。这篇文章主要介绍了vue3+vite中怎么解析markdown文档并实现代码高亮显示。这里选用的是markdown-it，代码高亮的库选用的是highlight.js。
# 「一、安装依赖库」
```
npm install markdown-it --save   // markdown-it 用于将markdown转换成html
npm install highlight.js -save   //用于代码高亮显示
```
# 「二、引入」
在main.ts文件中引入highlight.js及样式并创建一个自定义的全局指令
```
import hljs from 'highlight.js';
import 'highlight.js/styles/atom-one-dark.css' //样式
//创建v-highlight全局指令
Vue.directive('highlight',function (el) {
  let blocks = el.querySelectorAll('pre code');
  blocks.forEach((block)=>{
    hljs.highlightBlock(block)
  })
})
```
