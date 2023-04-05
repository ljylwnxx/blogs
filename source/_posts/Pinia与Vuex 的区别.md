---
title: Pinia 与 Vuex 的区别
date: 2023-04-05 19:58:26
tags: Vuex
categories: Vue
---
# 两者区别
1. Vuex的核心概念有state,getters,mutations,actions,moudles五个部分
Pinia的核心概念有state,getter,action三个部分
1.2 mutation 已被弃用。它们经常被认为是极其冗余的
1.3不再有嵌套结构的moudles，store的命名取决于它的定义方式，实现一种扁平架构
2. Vuex对state的修改推荐使用mutations中的方法进行修改,
Pinia直接对state进行修改
3. Pinia中 getter，action 也可通过 this 访问整个 store 实例
# main.js中
在main.js中，vuecli会自动将vuex挂载到app上，使用vite需要自己手动挂载pinia
```
import { createPinia } from 'pinia'  
const pinia = createPinia();  
app.use(pinia);
```
# store/index.js中
vuex有state,getter,mutation,action,moudle五个部分
```
import { createStore } from 'vuex'
import { receiveMessagePerson } from './modules/receiveMessagePerson'
export default createStore({
  state: {
    memberOrAdmin: "member",
    searchPerson: []
  },
  getters: {
  },
  mutations: {
    setMemberOrAdmin(context, data){
      context.memberOrAdmin = data
    },
  },
  actions: {
    getPerson(context){
      let userId = context.state.PersonId
      API.getAllInformation({userId}).then(res => {
        context.commit('setPerson',res.data)
      }).catch(err => {
        console.log(err);
      })
    }
  },
  modules: {
    receiveMessagePerson
  }
})
```
Pinia有state,getter,action三个部分
```
import { defineStore } from "pinia";
export const useTestStore = defineStore('Test', {
  state(){
    return{
      cuont: 0
    }
  },
  getters: {
    double(state){
      return state.cuont*2
    }
  },
  actions: {
    increase(){
      this.cuont++
    }
  }
})
```
# 在vue组件中
## vuex
vue2中：
```
this.$store.***
```
vue3中：
```
import { useStore } from 'vuex'
let store = useStore()
let *** = computed(() => store.state.***)
store.commit('**', **)
store.dispatch('**', **)
```
## vuex拆解
vue2中：
使用到mapState,mapGetter,mapMutations,mapActions方法,[_post/Vuex的namespaced属性.md]
vue3中：
## Pinia
```
import { useTestStore } from '../store/index';
let store = useTestStore()
store.**(state变量名)
store.**(getters变量名)
store.**(actions方法名)
```
省去了state,getters,commit,dispatch这些字段，更方便
## Pinia拆解
用到storeToRefs
```
import { storeToRefs } from 'pinia';
let { **, ** } = storeToRefs(store)    // state和getters的变量都可以拆解出来
let { ** } = store                   // actions的方法拆解出来
```