---
title: Vuex的namespaced属性
date: 2023-04-05 20:16:39
tags: Vuex
categories: Vue
---
举个例子试一下
# namespaced为false时
```
// 模块test
export default {
    state: {
        name: 'zb',
        age: 18
    },
    getters: {
        ageDouble(state){
            return (state.age)*2
        }
    },
    mutations: {
        addAge(state, num){
            state.age += num 
        }
    },
    actions: {
        addAge(context, num){
            context.commit('addAge', num)
        }
    }
}
// 在store/index.js中引入并注册
import test from './modules/test'   // 引入
**略**
modules: {
    test        // 注册
  }
```
namespaced属性默认为false，命名空间默认是关闭的，即模块内部的 action、mutation 和 getter 是注册在全局命名空间的。
## 使用模块test的state数据
this.$store.state.test.age
```
import { mapState } from 'vuex';
computed: {
    ...mapState({
      testName: state=>state.test.name,
      testAge: state=>state.test.age
    })
  },
this.testName
this.testAge
```
## 使用模块test的getters，mutation，action
调用模块test的getters，mutations，actions和store/index.js下的计算属性和方法没有区别，因为都在全局命名空间
### getters
this.$store.getters.ageDouble
```
import { mapGetters } from 'vuex';
computed: {
    // 起别名
    ...mapGetters({
      testAgeDouble: 'ageDouble'
    }),
    // 不起别名
    ...mapGetters(['ageDouble'])
  }, 
this.testAgeDouble
this.ageDouble
```
### mutations
this.$store.commit('addAge', 2)
```
import { mapMutations } from 'vuex';
methods:{
    // 起别名
    ...mapMutations({
      testAddAge: 'addAge'
    }),
    // 不起别名
    ...mapMutations(['addAge'])
  },
this.testAddAge(2)
this.addAge(2)
```
### actions
this.$store.dispatch('addAge', 2)
```
import { mapActions } from 'vuex';
methods:{
    // 起别名
    ...mapActions({
      testAddAge: 'addAge'
    }),
    // 不起别名
    ...mapActions(['addAge'])
  },
this.testAddAge(2)
this.addAge(2)
```
# namespaced为true时
```
export default {
    namespaced: true,
    state: {
        name: 'zb',
        age: 18
    },
    getters: {
        ageDouble(state){
            return (state.age)*2
        }
    },
    mutations: {
        addAge(state, num){
            state.age += num 
        }
    },
    actions: {
        addAge(context, num){
            context.commit('addAge', num)
        }
    }
}
// 在store/index.js中引入并注册
import test from './modules/test'   // 引入
**略**
modules: {
    test        // 注册
  }
```
namespaced属性设为true时，说明模块test拥有自己的命名空间，可以防止与其他模块的计算属性和方法混杂，看起来也更清晰。
## 使用模块test的state数据
同命名空间为false没有区别
this.$store.state.test.age
```
import { mapState } from 'vuex';
computed: {
    ...mapState({
      testName: state=>state.test.name,
      testAge: state=>state.test.age
    })
  },
this.testName
this.testAge
```
## 使用模块test的getters，mutation，action
### getters
this.$store.getters['test/ageDouble']
```
import { mapGetters } from 'vuex';
computed: {
    // 起别名
    ...mapGetters({
      testAgeDouble: 'test/ageDouble'
    }),
    // 不起别名
    ...mapGetters('test', ['ageDouble'])
  },
this.testAgeDouble
this.ageDouble
```
### mutations
this.$store.commit('test/addAge', 2)
```
import { mapMutations } from 'vuex';
methods:{
    // 起别名
    ...mapMutations({
      testAddAge: 'test/addAge'
    }),
    // 不起别名
    ...mapMutations('test', ['addAge'])
  },
this.testAddAge(2)
this.addAge(2)
```
### actions
this.$store.dispatch('test/addAge', 2)
```
import { mapActions } from 'vuex';
methods:{
    // 起别名
    ...mapActions({
      testAddAge: 'test/addAge'
    }),
    // 不起别名
    ...mapActions('test', ['addAge'])
  },
this.testAddAge(2)
this.addAge(2)
```
当命名空间开启之后，再使用模块test的计算属性和方法就要添加一个模块名的前缀。

