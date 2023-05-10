---
title: Vue3封装打字机效果组件
date: 2023-05-09 20:57:12
tags: vue3
categories: vue3
---
# 「摘要」
最近在做项目中有这样一个需求，就是需要实现一个打字机效果。当然打字机效果可以自己编写，使用css来实现，等等。为了不影响项目进度，便于日常好维护。决定找一找其他方式，经过查阅资料，翻看了各种资料之后，发现了TypeIt。

# 「typeit介绍」
typeit是一款轻量级打字机特效插件。该打印机特效可以设置打字速度，是否显示光标，是否换行和延迟时间等属性，它可以打印单行文本和多行文本，并具有可缩放、响应式等特点。

# 「安装typeit」
```
pnpm add typeit
npm install typeit
```
# 「配置项说明」
![](配置项说明.png)
# 「官网使用示例」
在官网我们就可以看出是如何使用的，通过.的方式来实现打字机效果。
```
<p id="hero"></p>
```
```
new TypeIt("#hero", {
  speed: 50,
  startDelay: 900,
})
  .type("the mot versti", { delay: 100 })
  .move(-8, { delay: 100 })
  .type("s", { delay: 400 })
  .move(null, { to: "START", instant: true, delay: 300 })
  .move(1, { delay: 200 })
  .delete(1)
  .type("T", { delay: 225 })
  .pause(200)
  .move(2, { instant: true })
  .pause(200)
  .move(5, { instant: true })
  .move(5, { delay: 200 })
  .type("a", { delay: 350 })
  .move(null, { to: "END" })
  .type("le typing utlity")
  .move(-4, { delay: 150 })
  .type("i")
  .move(null, { to: "END" })
  .type(' on the <span class="place">internet</span>', { delay: 400 })
  .delete(".place", { delay: 800, instant: true })
  .type('<em><strong class="font-semibold">planet.</strong></em>', {
    speed: 100,
  })
  .go();
```
根据上面我们掌握，了解到的信息，接下来我们使用typeit实现打字机效果，来实践一下。
首先我们使用vite初始化项目，初始化过程这里就省略不做详细说明。
# 「创建组件目录」
在components目录下创建一个typeit文件夹，再创建index.ts文件。
# 「封装组件」
接下来我们要在刚刚新创建的index.ts文件里，封装组件。
```
import TypeIt from 'typeit'
import { defineComponent, h } from 'vue'

export default defineComponent({
  name: 'TypeIt',
  props: {
    /** 打字速度，以每一步之间的毫秒数为单位 */
    speed: {
      type: Number,
      default: 200,
    },
    values: {
      type: Array,
      default: [],
    },
    className: {
      type: String,
      default: 'type-it',
    },
    cursor: {
      type: Boolean,
      default: true,
    },
  },
  render() {
    return h(
      'span',
      {
        class: this.className,
      },
      {
        default: () => [],
      },
    )
  },
  mounted() {
    new (TypeIt as any)(`.${this.className}`, {
      strings: this.values,
      speed: this.speed,
      cursor: this.cursor,
      breakLines: false,
      loop: true
    }).go()
  },
})
```
# 「使用组件」
```
<template>
  <div class="index-title">
        <h2 class="outline-none">
            <TypeIt :values="title" :cursor="false" :speed="150" />
        </h2>
  </div>
</template>

<script setup lang="ts">
import TypeIt from '@/components/typeit'

const title = [
    "陋室铭 -刘禹锡[唐代]",
    "山不在高，有仙则名。" ,
    ……
    ]
</script>
```