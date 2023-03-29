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
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
​
// https://vitejs.dev/config/
export default defineConfig({
  plugins: [vue()]
})
@vitejs/plugin-vue插件是对vue3语法做支持，如果要支持vue2，需要用vite-plugin-vue2
第一步，从vite中删除 @vitejs/plugin-vue配置，从package.json文件中也删除。
npm uninstall @vitejs/plugin-vue -D
第二步,安装vite-plugin-vue2依赖
npm install vite-plugin-vue2 -D
第三步,在vite.config.js文件配置vite-plugin-vue2
import { defineConfig } from 'vite'
import { createVuePlugin } from "vite-plugin-vue2";
​
// https://vitejs.dev/config/
export default defineConfig({
  plugins: [createVuePlugin()]
})
第四步，修改vue版本由3改为2版本
npm install vue@2 -S
第五步, 修改main.js，创建根vue实例写法改为vue2写法
import Vue from 'vue'
import App from './App.vue'
​
new Vue({
  render: h => h(App),
}).$mount('#app')
第六步，修改main.js完成后，修改App.vue文件代码为vue2格式代码
执行npm run dev，即可看到启动成功,代表此时vite已经支持vue2语法了，可以开始项目迁移工作了。
## 2.3复制原项目业务代码
第一步，复制原项目静态目录static下文件到vite项目public文件夹下
第二步，复制原项目index.html文件内容替换vite项目的index.html内容(注意本地静态资源引入的路径)替换后需要在body结束标签前添加 <script type="module" src="/src/main.js"></script>
第三步, 复制package.json中生产环境依赖到新vite项目,去除webpack相关配置依赖
第四步，复制原项目src业务文件代码，直接替换vite项目src文件
# 三、项目开发阶段报错处理
## 3.1第一个报错，Cannot find module 'node:path'
报错是由于项目所依赖的node版本与当前版本进行对比,发现了当前版本较低,所以出现了不兼容状况
## 3.2第二个报错，Failed to resolve import "./App" from "src/main.js
报错是由于引入App组件的时候没有带文件后缀 .vue, 所以未找到，此时有两种解决方案
1.手动添加 .vue后缀，但是项目这么庞大，很多地方都没有带后缀，全部改肯定不容易。
2.配置vite.config.js的extensions字段，来添加自动查找文件扩展名后缀。
采用第二种vite配置extensions扩展名，在vite.config.js里面添加resolve.extensions配置
resolve: {
    /** 暂时先加.vue, .js, .json **/
    extensions: [".vue", ".js", ".json"],
  }
## 3.3第三个报错，Cannot find module 'vue-template-compiler'
根据报错信息知道，找不到vue-template-compiler模块，因此我们安装一下即可
npm install vue-template-compiler -D
## 3.4第四个报错，require is not defined
这次启动项目后，命令行没有报错了，然后打开浏览器，发现页面白屏，打开控制台看到控制台报错
报错是使用reuire引入了一张图片，而vite不支持require，我们需要换一种引入方式来引入图片。
1. 第一种是采用import/from来引入，这种方式适合图片和所有模块，也是最符合规范，利用tree-shrink的。
2. 第二种是直接把图片提前压缩处理后放在public文件下，就可以通过根路径/xxx.png来访问到了。
3. 第三种使用vite提供的import.meta.glob()方法，但该方法返回的是异步的，适合配置懒加载动态路由。
这里我采用的是第二种方式，把xxx.png放在public目录下，require引入改成固定字符串 '/xxx.png' ,这样打包时就不用对图片做处理了，可以提高打包速度。
全局搜索一下require,把使用require引入图片的地方都改成绝对路径或者import/from引入。
## 3.5第五个报错，会发现此时接口请求报错（出现跨域问题，或者404）
这是因为vue中代理和vite中代理有区别，需要配置成vite规定的代理
server: {
    port: 3000,
    proxy: { 
      // 正则表达式写法
      '^/api': {
        target: 'http://localhost:xxxx/api', // 后端服务实际地址
        changeOrigin: true, //开启代理
        rewrite: (path) => path.replace(/^\/api/, '')
      }
    }
  }