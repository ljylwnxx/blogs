---
title: 微前端-qiankun
date: 2023-03-29 12:18:39
tags: 微前端
categories: 知识点
aubot: ljylwnxx
comments: false
toc: true
cover: [/images/qiankuncover.png]
banner: 
  type: img
  bgurl: [/images/qiankuncover.png]
---
qiankun 是一个基于 single-spa 的微前端实现库，旨在帮助大家能更简单、无痛的构建一个生产可用微前端架构系统。
# qiankun 的核心设计理念
1. 简单
由于主应用微应用都能做到技术栈无关，qiankun 对于用户而言只是一个类似 jQuery 的库，你需要调用几个 qiankun 的 API 即可完成应用的微前端改造。同时由于 qiankun 的 HTML entry 及沙箱的设计，使得微应用的接入像使用 iframe 一样简单。
2. 解耦/技术栈无关
微前端的核心目标是将巨石应用拆解成若干可以自治的松耦合微应用，而 qiankun 的诸多设计均是秉持这一原则，如 HTML entry、沙箱、应用间通信等。这样才能确保微应用真正具备 独立开发、独立运行 的能力。

# 为什么不是 iframe
一. 为什么不用 iframe，这几乎是所有微前端方案第一个会被 challenge 的问题。但是大部分微前端方案又不约而同放弃了 iframe 方案，自然是有原因的，并不是为了 "炫技" 或者刻意追求 "特立独行"。
如果不考虑体验问题，iframe 几乎是最完美的微前端解决方案了。
二. iframe 最大的特性就是提供了浏览器原生的硬隔离方案，不论是样式隔离、js 隔离这类问题统统都能被完美解决。但他的最大问题也在于他的隔离性无法被突破，导致应用间上下文无法被共享，随之带来的开发体验、产品体验的问题。
1. url 不同步。浏览器刷新 iframe url 状态丢失、后退前进按钮无法使用。
2. UI 不同步，DOM 结构不共享。想象一下屏幕右下角 1/4 的 iframe 里来一个带遮罩层的弹框，同时我们要求这个弹框要浏览器居中显示，还要浏览器 resize 时自动居中..
3. 全局上下文完全隔离，内存变量不共享。iframe 内外系统的通信、数据同步等需求，主应用的 cookie 要透传到根域名都不同的子应用中实现免登效果。
4. 慢。每次子应用进入都是一次浏览器上下文重建、资源重新加载的过程。
其中有的问题比较好解决(问题1)，有的问题我们可以睁一只眼闭一只眼(问题4)，但有的问题我们则很难解决(问题3)甚至无法解决(问题2)，而这些无法解决的问题恰恰又会给产品带来非常严重的体验问题， 最终导致我们舍弃了 iframe 方案。

# 特性
1. 基于 single-spa 封装，提供了更加开箱即用的 API。
2. 技术栈无关，任意技术栈的应用均可 使用/接入，不论是 React/Vue/Angular/JQuery 还是其他等框架。
3. HTML Entry 接入方式，让你接入微应用像使用 iframe 一样简单。
4. 样式隔离，确保微应用之间样式互相不干扰。
5. JS 沙箱，确保微应用之间 全局变量/事件 不冲突。
6. 资源预加载，在浏览器空闲时间预加载未打开的微应用资源，加速微应用打开速度。
7. umi 插件，提供了 @umijs/plugin-qiankun 供 umi 应用一键切换成微前端架构系统。

# 快速构建一个qiankun微前端应用
## 主应用的配置
1. 首先需要创建一个主应用和至少一个微应用
2. 然后在主应用中安装微前端库：qiankun，yarn add qiankun # 或者 npm i qiankun -S
3. 在主应用的入口处导入qiankun中的两个方法registerMicroApps和start方法
4. 调用registerMicroApps方法注册各个微应用，最后调用start方法来启动qiankun

## 微应用配置
1. 微应用不需要安装qiankun，但必须要在自己的入口处导出三个方法：
bootstrap：在微应用启动（初始化）的时候调用一次（只会调用一次）
mount：微应用渲染函数，每次微应用进入时都会调用
unmount：每次微应用切出或卸载时都会调用
**注意：**以上三个方法即使不做任何事也必须要导出，否则会报错
2. 为了保证每个微应用还能独立运行，还需要在入口处根据全局属性window.__ POWERED_BY_QIANKUN __ 来判断当前应用是由主应用启动还是自己独立运行，从而做出相应的逻辑处理。
+ 如果是独立运行则不需额外处理，只是按照原有逻辑进行渲染即可
+ 如果是由主应用启动的，则需将全局变量window.__ INJECTED_PUBLIC_PATH_BY_QIANKUN __ 的值赋给变量 __ webpack_public_path __
3. 如果是webpack等工具构建的应用（如vue或react等）则还需要给微应用的打包工具（webpack）增加一些相应的配置，一般需要配置output的library和libraryTarget。
4. 最后因为不同应用会部署在不同的域下，所以肯定还会有个跨域问题，一般会在服务器端再进行一个允许跨域的配置。

# 实践
## 主应用
1. 利用vue脚手架创建一个vue2.0项目：main-app（vue create main-app），建议选择手动模式可以将Router勾选上，并且需要使用history路由模式。
2. 打开项目main-app，安装qiankun（npm install qiankun --save）
3. 修改App.vue，新增一个subapp的router-link，to值为“/subapp”，并在router-view的下方新增一个id为vueContainer的div盒子（用于承载子应用）。
4. 修改views/Home.vue，将默认内容删除，并改为： This is a home page in qiankun-main（非必须，可根据自己喜好随便改动，也可使用默认内容）
5. 修改views/About.vue，将内容改为：This is an about page in qiankun-main（非必须，可根据自己喜好随便改动，也可使用默认内容）
6. 修改main.js，导入qiankun中的registerMicroApps和start两个方法，注册子应用并启动qiankun
下面看下修改后的完整代码（创建项目和安装qiankun的步骤就省略了）
```
<!-- App.vue -->
<template>
	<div id="app">
		<div id="nav">
			<router-link to="/">Home</router-link>
			<router-link to="/about">About</router-link>
			<router-link to="/subapp">sub-app</router-link> <!--新增部分-->
		</div>
		<router-view />
		<div id="vueContainer"></div><!--新增部分，用于承载子应用-->
	</div>
</template>
```
```
<!-- views/Home.vue -->
<template>
	<div class="home">This is a home page in qiankun-main</div>
</template>
```
```
<!-- views/About.vue -->
<template>
	<div class="home">This is an about page in qiankun-main</div>
</template>
```
```
<!-- main.js -->
import Vue from 'vue'
import App from './App.vue'
import router from './router'
<!-- ======================新增内容开始=============================== -->
import {registerMicroApps, start} from 'qiankun' //新增部分，导入qiankun中的两个方法
const apps = [
{
	name:'subApp', //子应用的名称
	entry:'//localhost:8081',//子应用的域名
	container:'#vueContainer',//承载子应用的容器，在上面App.vue中定义
	activeRule:'/subapp', // 被激活的子应用的路由
}
]
registerMicroApps(apps);//注册子应用
start();//启动qiankun
<!-- ======================新增内容结束=============================== -->
new Vue({
	router,
	render: h => h(App)
}).$mount('#app');
```
```
<!-- router/index.js -->
import Vue from 'vue'
import VueRouter from 'vue-router'
import Home from "../views/Home";
import About from "../views/About";
Vue.use(VueRouter)
const routes = [{
        path: '/',
        name: 'Home',
        // redirect: '/home',
        component: Home,
    },
    {
        path: '/about',
        name: 'About',
        component: About,
    },
]
<!-- 以下是修改后的代码 -->
const router = new VueRouter({
	mode:'history',
	base: '',
	routes
})
export default router;
```

## 微应用
微应用中主要需要修改的地方有：main.js、vue.config.js和router/index.js，其余页面部分根据自己喜好可改可不改，本文为了便于区分主子应用的内容将对Home.vue和About.vue页面进行微小的改动

1. 利用vue脚手架创建一个vue2.0项目：sub-app（vue create sub-app），建议选择手动模式可以将Router勾选上，并且需要使用history路由模式
2. 修改views/Home.vue，在原有内容的基础上新增语句：“This is a home page in sub-app”（根据个人喜好，可改可不改）
修改views/About.vue，将内容改为：“This is an about page in sub-app”（根据个人喜好，可改可不改）
3. 修改main.js（必需）
+ 将创建Vue实例的代码部分提取到一个函数render中，render函数接收一个参数props
+ 判断window.__ POWERED_BY_QIANKUN __，如果是从qiankun启动则将window. __ INJECTED_PUBLIC_PATH_BY_QIANKUN __ 的值赋值给 __ webpack_public_path __ ，否则直接调用render方法表示子应用是独立运行
+ 导出3个必需的方法bootstrap，mount和unmount；bootstrap函数体内容可为空但函数必须要导出。mount函数中调用render方法进行子应用渲染。unmount函数中将render方法中创建的vue实例销毁。
4. 修改router/index.js，指定base值为：“/vueChild”
5. 创建vue.config.js，在该文件中配置允许跨域：“ Access-Control-Allow-Origin：'*' ”，并配置webpack的output.library和output.libraryTarget

各部分完整代码如下：
```
<!--======================== views/Home.vue ====================-->
<template>
	<div class="home">
		<img alt="Vue logo" src="../assets/logo.png" />
		<h1 style="color:red;">This is a home page in sub-app</h1>
	</div>
</template>
```
```
<!--======================== views/About.vue ====================-->
<template>
	<div class="About">		
		This is an about page in sub-app
	</div>
</template>
```
```
<!--  main.js -->
import Vue from 'vue'
import App from './App.vue'
import router from './router'
let instance = null; //设置全局变量，用于保存或销毁Vue实例
function render(props = {}){
	const { container } = props
	instance = new Vue({
		router,
		render: h => h(App)
	}).$mount(container ? container.querySelector("#app") : "#app");//用于限定当前上下文下的#app，防止与主应用中的#app冲突
}
if(window.__POWERED_BY_QIANKUN__){
	__webpack_public_path__ = window.__INJECTED_PUBLIC_PATH_BY_QIANKUN__
}else{
	render();
	console.log('子应用独立运行')
}
export async function bootstrap(props){
	console.log('这里暂时可以什么都不用做，但方法必须要导出')
}
export async function mount(props){
	render(props);//从qiankun启动
}
export async function unmount(props){
	instance.$destroy();//销毁子应用实例
}
```
```
<!-- vue.config.js -->
module.exports = {
	lintOnSave: false,
	devServer:{
		port:8081,
		headers:{
			"Access-Control-Allow-Origin": "*"
		}
	},
	configureWebpack:{
		output:{
			library:'subApp',
			libraryTarget:'umd'
		}
	}
}
```
```
<!-- router/index.js -->
// ...原有代码省略
//修改后的代码
const router = new VueRouter({
	mode:'history',
	base:'/subapp',
	routes
})
```