---
title: 【vite+vue3+Ts+element-plus1】
date: 2023-04-26 11:51:29
tags: vue3
---
# 使用vite初始化项目
首先我们使用vite来初始化项目，vite支持使用npm、yarn和pnpm来初始化项目，本文采用npm的方式。
```
// 使用npm
npm create vite@latest
// 使用yarn
yarn create vite
// 使用pnpm
pnpm create vite
```
输入命令之后会提示我们输入项目名，如果我们已经建好项目，并且是在项目根目录执行的命令的话，只需输入一个点表示在当前目录创建就行，需要注意的是当前项目文件夹必须是空文件夹，否则vite会提示你是否清空已存在的文件继续。
如果还没建项目，是在父目录执行的命令，则需输入要创建的项目名
之后会让我们选择使用的框架和选择使用js还是ts，我们选择vue和ts。
之后我们按照提示命令进入项目目录执行命令安装依赖，运行项目即可。
之后我们把一些默认的没用的文件给干掉，再在相关引用的地方把引用给清空。
![](qingkong.png)
再把APP.vue里面的内容给清空，这个时候整个项目就是干干净净的了。
![](qingkongapp.png)
# vscode推荐插件
使用vscode的用户注意了，之前我们开发vue可能使用的是vetur插件，但是从vue3开始，官方已经推荐使用Volar了，所以我们需要把旧的插件给禁用或者直接卸载掉。
![](vetur.png)
再去扩展里搜索volar进行安装就可以了。
![](volar.png)
# vite配置目录别名
在项目开发中，我们通常会引用其他目录的文件，但如果文件不在一个目录的话，就会使用../的方式去查找，如果层级多的话很不方便，所以我们通常会给一些常用目录配置一个别名，比如下面我们就在vite中给src目录配置一个‘@’别名
首先我们需要安装node的类型声明文件，要不然ts可能会报警告。
```
yarn add @types/node -D
```
之后我们在vite.config.ts中进行相关配置
```
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import path from 'path'                   // 增加此行代码
const pathSrc = path.resolve(__dirname, 'src')
// https://vitejs.dev/config/
export default defineConfig({
  plugins: [
    vue(),
  ],
  resolve: {                              
    alias: {              // 增加此行代码                
      '@': pathSrc,
    },                                
  },  
})
```
之后还要在tsconfig.json中增加以下配置
```
 "paths": {
      "@/*": ["./src/*"]
    }
```
# 重置默认样式
我们这里直接在assets/style文件夹下新建一个normalize.css，写入重置样式。
然后在main.ts中引入即可
```
// main.ts
import '@/assets/style/normalize.css'
```
# vue-router
首先我们来安装一下vue-router
```
yarn add vue-router@4
```
之后在src目录下新建router/index.ts，写入一些基本路由。
这里需要注意的是，vue-router指定路由模式从字符串变为了使用方法，history是createWebHistory(地址栏不带#)，hash是createWebHashHistory（地址栏带#）。
这时候我们可能还会发现，编辑器中ts会报找不到vue文件相关类型声明的错误。
![](router.png)
## 解决办法：我们可以在根目录创建一个env.d.ts，在里面写入以下内容
```
/// <reference types="vite/client" />
declare module '*.vue' {
  //引入vue模块中ts的方法
  import type { DefineComponent } from 'vue'
  // 定义vue組件以及类型注解
  const component: DefineComponent<{}, {}, any>
  export default component
}
```
之后还要记得在tsconfig.ts中引入。
![](ts.png)
现在报错没有了，我们还需要在main.ts中进行引入
```
import router from './router'
createApp(App).use(router).mount('#app')
```

