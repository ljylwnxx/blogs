---
title: webpack迁移vite
date: 2023-03-29 21:41:53
tags: webpack
categories: 知识点
---
# 一、迁移流程
1.先创建新的vite项目
2.新版vite项目默认是支持vue3的，需要把vue改成vue2版本后配置vite-plugin-vue2插件来支持vue2
3.把项目代码改成vue2写法，确保新vite项目可以正常运行vue2
4.把原webpack项目生产环境依赖复制到vite项目，剔除掉webpack相关的插件依赖
5.复制原项目src文件代码和其他业务相关代码到新vite项目。
6.新vite项目配置开发环境启动命令，根据报错信息来进行调整。
7.在测试开发和打包环境都没问题后，替换原先的项目。
# 二、迁移业务代码到vite项目
## 2.1创建新的vite项目
由于原先项目没有用ts，所以创建项目不选ts版本，包管理工具也依然选择是npm。
npm init vite@latest my-vue-app -- --template vue
创建完成后，使用vs code打开，打开命令行，执行npm i安装依赖
npm i
安装依赖完成后，使用npm run dev启动项目
此时基本的vite2+vue3项目已经启动成功了，但此时vite支持的还是vue3版本的，我们需要让vite支持vue2版本。
## 2.2配置vite支持vue2
此时打开vite.config.js,里面的代码为
```
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
// https://vitejs.dev/config/
export default defineConfig({
  plugins: [vue()]
})
```
@vitejs/plugin-vue插件是对vue3语法做支持，如果要支持vue2，需要用vite-plugin-vue2
第一步，从vite中删除 @vitejs/plugin-vue配置，从package.json文件中也删除。
npm uninstall @vitejs/plugin-vue -D
第二步,安装vite-plugin-vue2依赖
npm install vite-plugin-vue2 -D
第三步,在vite.config.js文件配置vite-plugin-vue2
```
import { defineConfig } from 'vite'
import { createVuePlugin } from "vite-plugin-vue2";
// https://vitejs.dev/config/
export default defineConfig({
  plugins: [createVuePlugin()]
})
```
第四步，修改vue版本由3改为2版本
npm install vue@2 -S
第五步, 修改main.js，创建根vue实例写法改为vue2写法
```
import Vue from 'vue'
import App from './App.vue'
new Vue({
  render: h => h(App),
}).$mount('#app')
```
第六步，修改main.js完成后，修改App.vue文件代码为vue2格式代码
执行npm run dev，即可看到启动成功,代表此时vite已经支持vue2语法了，可以开始项目迁移工作了。
## 2.3复制原项目业务代码
第一步，复制原项目静态目录static下文件到vite项目public文件夹下
第二步，复制原项目index.html文件内容替换vite项目的index.html内容(注意本地静态资源引入的路径)替换后需要在body结束标签前添加 <script type="module" src="/src/main.js"></script>
第三步, 复制package.json中生产环境依赖到新vite项目,去除webpack相关配置依赖
第四步，复制原项目src业务文件代码，直接替换vite项目src文件
# 三、项目开发阶段报错处理