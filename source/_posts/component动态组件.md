---
title: component动态组件
date: 2023-03-23 21:27:48
tags: vue
categories: 知识点
---
# 一、什么是动态组件？
定义:多个组件挂载到同一个组件上，通过参数动态的切换不同组件就是动态组件。
书写形式：<component :is="componentName"></component>
内置组件：
component：是vue里面的一个内置组件。作用是：配合is动态渲染组件。
vue内置的组件还包括：
transition：作为单个元素/组件的过渡效果。
transition-group：作为多个元素/组件的过渡效果。
keep-alive：包裹动态组件时，会缓存不活动的组件实例，而不是销毁它们。
slot：作为组件模板之中的内容分发插槽。
# 二、使用方式
通过使用<component>元素动态的绑定到它的is特性，来实现动态组件的切换。
如果is匹配不到相应的组件的时候是不尽行任何dom元素的渲染的。
1.在不同组件之间进行动态切换
component动态组件就是通过控制currentTabComponent来切换不同的组件。
```
<div @click="reload">点击切换</div> 
<component :is="currentTabComponent"></component>
<script>
import childOne from './childOne'
import childTwo from './childTwo'
export default {
    componets:{
        childOne,
        childTwo
    },
    data(){
        currentTabComponent: 'childOne'
    },
    methods:{
        reload(){
            this.currentTabComponent = 'childTwo'
        }
    }
}
</script>
```
# 三、动态组件的缓存
使用动态组件来回切换时，组件是要被销毁的，若不想让数据销毁可以使用<keep-alive>，它可以包裹动态组件，会缓存不活动的组件实例，这样就不会被销毁。和 <transition> 相似，<keep-alive> 是一个抽象组件：它自身不会渲染一个 DOM 元素，也不会出现在父组件链中。
<keep-alive></keep-alive>包裹动态组件时，会缓存不活动的组件实例,主要用于保留组件状态或避免重新渲染。
大白话:比如有一个列表和一个详情，那么用户就会经常执行打开详情=>返回列表=>打开详情…这样的话,列表和详情都是一个使用频率很高的页面，那么就可以对列表组件使用<keep-alive></keep-alive>进行缓存，这样用户每次返回列表的时候，都能从缓存中快速渲染，而不是重新渲染。

```
<keep-alive>
    <component :is="componentName"></component>
</keep-alive>
```
