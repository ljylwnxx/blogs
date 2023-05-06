---
title: Vue3 + Vite + TS项目引入iconfont图标（Svg方式）
date: 2023-05-06 11:20:51
tags: vue3
categories: vite
---
# 🍻 前言
每一个项目都避免不了使用各种各样的图标，如果我们使用了 UI 组件库，比如说 ELement 等，那么组件库有一些封装好的图标供我们使用。但是项目是多变的和复杂的，组件库提供的图标很多时候不能满足需求，这个时候就需要我们自己引入想要的图标了。
今天介绍的便是如何将 iconfont 阿里图标库的图标引入到我们的 Vue3 项目中来！
# 实现步骤
## 安装
```
npm install vite-plugin-svg-icons -D
```
# 引入 iconfont
## 封装 svg-icon 组件
我们在 iconfont 官网上可以看到给出了 symbol 引用示例，代码如下：
```
<svg class="icon" aria-hidden="true">
    <use xlink:href="#icon-xxx"></use>
</svg>
```
上段代码就是图标的具体使用方式，如果我们每次都按照上面的方式使用，那么无疑是很麻烦的，我们不妨将上面的代码封装为一个组件。在需要用到图标的地方直接引入我们的组件即可了。
在 components 目录下新建 SvgIcon 目录，然后新建 SvgIcon.vue 文件。
代码如下：
```
// src/components/SvgIcon/SvgIcon.vue
<template>
  <svg :class="svgClass" aria-hidden="true">
    <use :xlink:href="iconClassName" :fill="color" />
  </svg>
</template>
<script setup lang="ts">
import { computed } from 'vue';
const props = defineProps({
  iconName: {
    type: String,
    required: true
  },
  className: {
    type: String,
    default: ''
  },
  color: {
    type: String,
    default: '#409eff'
  }
});
// 图标在 iconfont 中的名字
const iconClassName = computed(()=>{
  return `#${props.iconName}`;
})
// 给图标添加上类名
const svgClass = computed(() => {
  if (props.className) {
    return `svg-icon ${props.className}`;
  }
  return 'svg-icon';
});
</script>
<style scoped>
.svg-icon {
  width: 1em;
  height: 1em;
  position: relative;
  fill: currentColor;
  vertical-align: -2px;
}
</style>
```
组件封装好后我们还需要全局注册一下，不然每次引用图标的时候还得单独引入一次该组件。所以我们就在 main.ts 里面全局注册一下。
代码如下：
```
// main.ts
import { createApp } from 'vue'
import App from './App.vue'
import SvgIcon from './components/SvgIcon/SvgIcon.vue'

const app = createApp(App)
app.component('SvgIcon', SvgIcon);
app.mount('#app')
```
## 使用iconfont
接下来我们就需要去 iconfont 官网了，新建一个 iconfont 资源库，存放自己的 iconfont。
我们选中 symbol 模式，这里我介绍三种引入方式。
第一种：
直接在线引入官网提供的在线 js 地址，我们直接以 script 标签的形式引入即可。这种方式最为简单，但是也有不好的一点，需要用户有网络环境，而且得保证 iconfont 网站没有崩掉。
第二种：
直接下载至本地，我们从官网直接将代码下载下来，然后放到我们项目中引用，也是可以的。不过这种方式稍显麻烦，每次更新图标库之后都得重新下载一遍。
第三种：
这也是我比较喜欢的方式，也就是将在线地址中的代码直接复制粘贴到我们项目中来，这种方式最为简单，每次更新图标库之后只需要重新复制一下代码即可。这里我们也将采用这种方式。
具体使用：
在项目 assets 目录下新建 iconfont 目录，在该目录下新建 iconfont.js 文件，然后将 iconfont 在线地址中提供的代码全部复制过来。
然后在 main.ts 中引入 iconfont.js。
代码如下：
```
import { createApp } from 'vue'
import App from './App.vue'
import SvgIcon from './components/SvgIcon/SvgIcon.vue'
import './assets/iconfont/iconfont.js';
const app = createApp(App)
app.component('SvgIcon', SvgIcon);
app.mount('#app')
```
## 具体使用
引入 iconfont 得过程非常简单，主要分为了以下两步：
+ 封装 svg-icon 组件
+ 引入 iconfont 生成的 js 代码

接下来我们就实际使用试试，我们就直接在 App.vue 中引入几个图标试试。
代码如下：
```
<template>
  <img alt="Vue logo" src="./assets/logo.png" />
  <svg-icon iconName="icon-gongzuoleixing"></svg-icon>
  <svg-icon iconName="icon-yulan" className="yulan"></svg-icon>
</template>

```
可以看到我们的图标已经可以使用了，其中 iconName 属性值就是我们在 iconfont 网站复制的 iconfont 的名称代码。如果想要该颜色或大小，可以自行传入一个类或者 color 属性。
除此之外，如果你有自己下载好的 svg 文件，那么也是可以通过上面方式引入的，只需要将 iconName 改为你自己本地的 svg 名称即可。

# 图标自动管理（必看）
即使使用了symbol方式，当设计小姐姐新增图标时，我们还是无法避免重新生成图标代码。那么有没有更优雅的解决方案呢？
答案是有的。svg-sprite-loader + require.context。

# 总结
+ 支持多色图标了，不再受单色限制。
+ 支持丰富的css属性进行定制。
+ 兼容性较差，支持 ie9+,及现代浏览器。
+ 浏览器渲染svg的性能一般，还不如png。