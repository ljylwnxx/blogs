---
title: Vue3封装打字机效果组件
date: 2023-05-09 20:57:12
tags: vue3
categories: vue3
---
# 「typeit 介绍」
typeit是一款轻量级打字机特效插件。该打印机特效可以设置打字速度，是否显示光标，是否换行和延迟时间等属性，它可以打印单行文本和多行文本，并具有可缩放、响应式等特点。

# 「安装typeit」
```
pnpm add typeit
npm install typeit
```
# 「配置项说明」
![](配置项说明.png)
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