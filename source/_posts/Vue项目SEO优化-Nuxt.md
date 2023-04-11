---
title: Vue项目SEO优化-Nuxt
date: 2023-04-10 22:12:36
tags: vue
categories: 性能优化
cover: [/images/cover.png]
banner: 
  type: img
  bgurl: [/images/cover.png]
---
# 为什么SPA需要SEO
通常我们用前端框架（这里以Vue为例）产出的代码一般是这样的：
```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>title</title>
</head>
<body>
  <div id="app"></div>
  <script src="index.js"></script>
</body>
</html>
```
页面实际展示的内容（用户实际看到的内容）都是通过js渲染出来的，所以搜索引擎爬虫在对我们的页面进行爬取的时候，拿到的也是上面的html代码。虽然我们可以通过在<head></head>中添加<meta/>标签来添加一些描述信息，但是远远不够，我们实际展示的内容搜索引擎爬虫还是拿不到，所以就需要对项目进行SEO优化。
# 关于Nuxt.js
## 官方介绍
从头搭建一个服务端渲染的应用是相当复杂的。幸运的是，我们有一个优秀的社区项目Nuxt.js这让一切变得非常简单。Nuxt是一个基于Vue生态的更高层的框架，为开发服务端渲染的Vue应用提供了极其便利的开发体验。更酷的是，你甚至可以用它来做为静态站生成器。
## 简单的理解：
代替浏览器的工作,笼统理解就是在created时的请求数据和页面渲染，第二点是当作静态文件服务器，把渲染好的页面返回给用户。
## 优势：
+ 纯静态文件，访问速度快
+ 对比SSR,不涉及服务器负载方面问题
+ 静态网页不宜遭到黑客工具，安全性高
## 服务端渲染和客户端渲染
1. 浏览器（客户端）通过AJAX向服务端（java servlet）发送http请求数据接口
2. 服务端将获取的接口数据封装成JSON，响应给浏览器
3. 浏览器拿到JSON就进行渲染html页面，生成DOM元素，然后将页面展示给用户
![](服务端渲染.png)
![](客户端渲染.png)
## Nuxt.js工作原理
1. 浏览器(客户端)发送http请求到Node.js服务端。
2. 部署在Node.js的应用Nuxt.js接收到浏览器请求，它会去请求后台服务端。
3. 后台接口服务端会响应JSON数据，Nuxt.js获取数据后进行服务端渲染成html。
4. 然后Nuxt.js将html页面响应给浏览器
5. 浏览器直接将接收到html页面进行展示
![](nuxt渲染.png)
## 操作
### 首先就是安装NUXT这个简单，大家按照官网的操作步骤就可以了。
### 目录结构
```
.nuxt
1. assets //资源目录 组织为编译的静态资源 大家都懂
      image.png
2.components // 组织 Vue.js 组件，NUXT不会扩展增强该目录下的组件，这意味着组件不能使用 
3.asyncData 方法
4.layout // 布局目录 如果没有额外配置，目录不能重命名--不建议改名
5.middleware // 用于存放应用的中间件
6.node_modules 
7.pages // 页面目录用于组织应用路由及视图，NUXT会读取该目录下所有.vue文件并自动生成路由（路由文件在 .nuxt/router.js）
      index.vue
8.plugins // 插件目录 用于组织那些需要在跟vue.js应用，实例化之前需要运行的Javascript插件
9.static // 用于存放应用的静态文件，此类文件不会被NUXT.js调用Webpack 进行构建编译处理。服务器启动的时候，该目录下的文件会映射至应用的根路径  `/`  下。例如 /static/root.txt 映射至 /root.txt
10.store // Nuxt.js 框架集成了 [Vuex 状态树] 的相关功能配置，在  `store`  目录下创建一个  `index.js`  文件可激活这些配置。 新建一个index.js 这事就算妥了
11.nuxt.config.js // 文件用于组织 Nuxt.js 应用的个性化配置，以便覆盖默认配置 不要gai'ming'zi
12.package.json // 用于描述应用的依赖关系和对外暴露的脚本接口
```
### NUXT开发中需要注意哪些。
#### 静态资源的引入和路由
```
//引入静态资源 和 跳转路由
<template>
  <!--我们不需要去配置 @去指向根目录  NUXT中可以直接使用 -->
  <img src="~/assets/image.png" />
  <!--NUXT中我们不需要去写路由表  👇 '/'指的是 page/index.vue 其它语法和日常无恙  -->
  <nuxt-link :to="{name:'userPage',query:{id:'111'}}">个人中心</nuxt-link>
</template>
```
#### asyncData
Nuxt.js扩展了Vue.js 增加了asyncData的方法，这样我们可以在渲染组件之前异步获取数据，asyncData方法会在组件（只限于页面组件）每次加载之前被调用，它可以在服务端或路由更新之前被调用，在这个方法被调用的时候，第一个参数context被设定为当前页的上下文对象。
```
async asyncData({ params }){  // params 就是传进来的值
//asyncData 函数去执行我们的异步操作 当我们获取到接口返回的内容是 此时我们Vue还没有实例化 所以this获取不到  我们通过返回 方法 去获取  因此标签内如果需要展示内容  {{info.XXX}}
	const data = await $axios.$get('/api/user')
	return {data}
}
```
#### @nuxtjs/axios
我们在asyncData中调用接口，在Nuxt.js官方提供了@nuxtjs/axios模块，此模块还包含了axios、@nuxtjs/proxy（解决异步,进行代理转发）模块。
```
// 1.安装@nuxtjs/axios：
npm install @nuxtjs/axios
// 2.在nuxt.config.js中配置axios
module.exports={
	modules:['@nuxtjs/axios']
}
```
#### 中间件
中间件允许您定义一个自定义函数运行在一个页面或一组页面渲染之前。可用于权限判断，有权限才可访问对应页面
中间件应放置在middleware/ 目录。文件名的名称将成为中间件名称（middleware/auth.js将成为auth中间件）
```
//创建权限中间件
//在 middleware/ 下创建 auth.js 文件，其中auth就是中间件的名称。
//一个中间件接收content作为第一个参数
export default({ store, redirect }) => {
	if(/* 没有token */){
		return redirect('/')
	}
}
```
# Nuxt对Vue项目首页进行SEO优化
Nuxt.js 集成了以下组件/框架，用于开发完整而强大的Web 应用：
+ Vue 2
+ Vue-Router
+ Vuex(当配置了Vuex状态树配置项时才会引入)
+ Vue服务器端渲染 (排除使用mode:'spa')
+ Vue-Meta
另外，Nuxt.js使用Webpack和vue-loader 、babel-loader来处理代码的自动化构建工作（如打包、代码分层、压缩等等）。
## 具体步骤
### 新建项目
因为是对项目的一部分进行改造，所以新建了一个项目。
新创建的项目目录大致如下：
```
+ assets:静态资源
+ components:组件，该文件夹下的组件使用时无需引入，类似于已经全局注册了，这些组件无法使用asyncData。
+ layouts:用于组织应用的布局组件，我理解类似于Vue项目中的App.vue
+ middlwware:应用的中间件，这个没有用到
+ node_modules:依赖包
+ pages:页面，Nuxt.js 框架读取该目录下所有的 .vue 文件并自动生成对应的路由配置。在这个目录里使用其他类型的文件在npm run generate时可能会报错
+ plugins:插件目录。用于组织需要再根Vue.js应用实例化之前需要运行的JavaScript插件。在任何 Vue 组件的生命周期内，只有 beforeCreate 和 created 这两个方法会在 客户端和服务端被调用。其他生命周期函数仅在客户端被调用。
+ static:静态文件目录。
静态文件目录 static 用于存放应用的静态文件，此类文件不会被 Nuxt.js 调用 Webpack 进行构建编译处理。服务器启动的时候，该目录下的文件会映射至应用的根路径 / 下。
store
+ Vuex状态树。
Nuxt.js 框架集成了 Vuex 状态树 的相关功能配置，在 store 目录下创建一个 index.js 文件可激活这些配置。
+ nuxt.config.js
nuxt.config.js 文件用于组织 Nuxt.js 应用的个性化配置，以便覆盖默认配置。
+ 其他
.babelrc、.editorconfig、package.json、package-lock.json等，和其他项目中同名文件功能相同。
+ 别名
~或@ => srcDir
~~或@@ => rootDir
```
在vue模板中, 如果需要引入assets或者static目录, 使用~/assets/your_image.png和~/static/your_image.png方式。
### 迁移文件
把老项目首页及宣传页的代码以及依赖的静态文件，还有一些eslint的配置，webpack配置等，能用的都拿过来，然后修改引用路径，删除不需要的代码。
下面是目录的对应关系：
```
+ assets:css、icon、font等静态资源
+ components:首页及宣传页用到的组件
+ layouts:App.vue
+ middlwware
+ node_modules:依赖包
+ pages:首页及宣传页涉及到的页面。
+ plugins:用到的各种插件。
      + axios封装（没有用Nuxt.js默认的@nuxtjs/axios）
      + elementUI
      + 其他一些用到的插件（video、zepto等）
      + 工具函数（这里有疑问，是不是应该放在assets下面）
+ static:.ico文件
+ store:vuex相关文件
+ nuxt.config.js:nuxt.config.js 文件用于组织 Nuxt.js 应用的个性化配置，以便覆盖默认配置。
+ 其他:.babelrc、.editorconfig、package.json、package-lock.json等，和其他功能相同。
```
### 修改配置
这部分是重点，因为Nuxt很多东西其实在一开始就都配置好了，比如像它的目录结构，官方都建议一般不要修改。如果想修改配置的话，必须在nuxt.config.js中进行修改。Nuxt关于配置的文档很多，这里直说我用到的部分。
```js
import webpack from 'webpack'
export default {
  ssr: false, // 是否启用服务端渲染。默认为false，创建时可选择
  target: 'static', // server：用于服务器端渲染，static：用于静态网站
  root: {
    base: '/page/' // 应用程序的基本URL，如果在此路径下提供了整个应用程序，则应该使用root.base。使用之后，访问***.com/home路由应该使用***.com/page/home
  },
  head: { // 一些静态资源的引入，填写头信息
    title: 'nuxt-demo',
    meta: [
      { charset: 'utf-8' },
      { name: 'viewport', content: 'width=device-width, initial-scale=1' },
      { hid: '', name: 'description', content: '' }
    ],
    script: [
      {
        src: 'https://cdn.bootcdn.net/ajax/libs/zepto/1.2.0/zepto.js'
      }
    ]
  },
  css: [ // 全局使用的css
    'element-ui/lib/theme-chalk/index.css',
    '~/assets/css/common.css',
    '~/assets/css/element-variables.scss',
  ],
  plugins: [ //  使用插件
    {src: '~plugins/element-ui'},
    {
      src: '~plugins/log.js',
      mode: 'client' // client或server：文件仅包含在客户端或服务器端，如果代码中使用了window或document等浏览器端才有的API，只能使用client
    },
    {
      src: '~plugins/route.js',

 mode: 'client'
    }
  ],
  components: true, // 自动引入组件
  buildModules: [ // 某些模块仅在开发和构建期间需要。使用buildModules有助于加快生产启动速度，并大大减少node_modules生产部署的规模。
    '@nuxtjs/eslint-module'
  ],
  build: { // Nuxt.js 允许你根据服务端需求，自定义 webpack 的构建配置。
    transpile: [/^element-ui/], // 使用Babel与特定的依赖关系进行转换。应该是element-ui使用了ES6语法，所以需要进行转化
    publicPath: '', // 静态资源引用路径（通常使用CND地址）
    plugins: [ // 配置webpack插件  nuxt使用的webpack版本比较旧，有一些新功能无法使用
      new webpack.DefinePlugin({
        __ENV__: JSON.stringify(process.env.ENV || 'dev')
      })
    ]
  }
}
```
### 项目部署
由于该项目与其他项目共用一个域名：www.###.com，需要保证另一个项目也可用。所有在部署的时候把Nuxt的项目放在了www.###.com/page目录下
在server中添加location /page的配置项
让根路径重定向到www.###.com/page/home
对旧项目已有的一些访问路径进行兼容
首页项目和旧项目（主站）的相互跳转（登录登出跳转等）
### 常见问题
一、document is not defined or window is not defined
解决方法：
此问题产生是由于在服务端渲染时使用了window或document等DOM相关的API，常见的原因有两个：
+ 页面中生命周期使用错误，比如在created中使用了window
nuxt在服务端执行时会执行Vue生命周期中的beforeCreate和created，如果在这两个生命周期里使用了DOM相关的API是获取不到的，构建时会报错。
+ 第三方插件使用了window或document等DOM相关的API
Nuxt.js 允许在运行Vue.js应用程序之前执行js插件。这在您需要使用自己的库或第三方模块时特别有用。但是要注意插件是否在客户端和服务端都需要，还有就是插件中是否用到了DOM相关的API（会导致构建时报错）
解决方法：
+ 正确使用生命周期，在任何 Vue 组件的生命周期内， 只有 beforeCreate 和 created 这两个方法会在 客户端和服务端被调用。其他生命周期函数仅在客户端被调用。
这里获取不到window或document等API，可以放在mounted中执行。
+ 使用第三方插件时：
```
plugins: [
  {
    src: '~plugins/log.js',
    mode: 'client'
  }
]
```
添加mode: 'client'使第三方插件仅在客户端渲染时使用。
二、NuxtLink无法打开新页面
Nuxtjs中在进行路由导航时提供了<nuxt-link to="/"></nuxt-link>，但是此标签有一个问题：无法在新窗口打开链接，建议改为a标签，既能满足需求，也利于搜索引擎爬取相关链接。
三、注册全局方法
有时想在整个项目中使用函数或值，在Vue项目中我们有可能把方法挂在在Vue实例上，但是官方文档中明确提出：
请勿全局使用Vue.use()，Vue.component()，也不要在此功能内专门用于Nuxt注入的Vue中插入任何内容。这将导致服务器端内存泄漏。
另外也为我们提供了一种方法（inject(key, value)）来注册全局方法（变量）：
```
// plugins/hello.js
export default ({ app }, inject) => {
  // Inject $hello(msg) in Vue, context and store.
  inject('hello', msg => console.log(`Hello ${msg}!`))
}
```
```
// nuxt.config.js
export default {
  plugins: ['~/plugins/hello.js']
}
```
然后就可以在页面、组件、插件中使用了。
```
// a.vue
export default {
  mounted() {
    this.$hello('mounted')
    // will console.log 'Hello mounted!'
  },
  asyncData({ app, $hello }) {
    $hello('asyncData')
    // If using Nuxt <= 2.12, use 👇
    app.$hello('asyncData')
  }
}
```
四、Nuxt支持在页面中加入head信息。
```
export default {
  data () {
    return {}
  },
  created () {},
  ...,
  head () {
    return {
      title: '',
      meta: [
        { hid: '', name: '', content: ''}
      ]
    }
  }
}
```
五、路由相关
Nuxt.js 依据 pages 目录结构自动生成 vue-router 模块的路由配置。
基础路由
pages/
--| user/
-----| index.vue
-----| one.vue
--| index.vue
那么，Nuxt.js 自动生成的路由配置如下：
```
router: {
  routes: [
    {
      name: 'index',
      path: '/',
      component: 'pages/index.vue'
    },
    {
      name: 'user',
      path: '/user',
      component: 'pages/user/index.vue'
    },
    {
      name: 'user-one',
      path: '/user/one',
      component: 'pages/user/one.vue'
    }
  ]
}
```
嵌套路由
你可以通过 vue-router 的子路由创建 Nuxt.js 应用的嵌套路由。
创建内嵌子路由，你需要添加一个 Vue 文件，同时添加一个与该文件同名的目录用来存放子视图组件。
文件路径：
pages/
--| users/
-----| _id.vue // 以下划线作为前缀的Vue文件：动态路由，做SSR的时候可能会用到
-----| index.vue
--| users.vue
Nuxt.js 自动生成的路由配置如下：
```
router: {
  routes: [
    {
      path: '/users',
      component: 'pages/users.vue',
      children: [
        {
          path: '',
          component: 'pages/users/index.vue',
          name: 'users'
        },
        {
          path: ':id',
          component: 'pages/users/_id.vue',
          name: 'users-id'
        }
      ]
    }
  ]
}
```
六、常用命令：
nuxt
启用一个热加载的Web服务器（开发模式）localhost:3000
nuxt build
利用webpack编译应用，压缩JS和CSS资源（发布用）。
nuxt start
以生产模式启动一个Web服务器 (基于dist目录，需要先执行nuxt build)。
nuxt generate
编译应用，并依据路由配置生成对应的HTML文件 (用于静态站点的部署)。