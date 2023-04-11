---
title: Vuex
date: 2023-04-05 19:49:07
tags: Vuex
categories: Vue
cover: [/images/vuexcover.png]
banner: 
  type: img
  bgurl: [/images/vuexcover.png]
---
Vuex它是一个程序里面的状态管理模式，它是集中式存储所有组件的状态的小仓库，并且保持我们存储的状态以一种可以预测的方式发生变化。
# 第一步，了解Vuex
## 问题假设:
如果你的项目里有很多页面（组件/视图），页面之间存在多级的嵌套关系，此时，这些页面假如都需要共享一个状态的时候，此时就会产生以下两个问题：
1. 多个视图依赖同一个状态
2. 来自不同视图的行为需要变更同一个状态
## 解决以上方法的方案:🤪
1. 对于第一个问题:
+ 假如是多级嵌套关系，你可以使用父子组件传参进行解决；
+ 对于兄弟组件或者关系更复杂组件之间，就很难办了，虽然可以通过各种各样的办法解决，可实在很不优雅，而且项目是越做越大，代码就会变得越多。
2. 对于第二个问题：
+ 你可以通过父子组件直接引用，或者通过事件来变更或者同步状态的多份拷贝，这种模式很脆弱，往往使得代码难以维护，而且同样会让代码就会变得越多。
## 不如换一种思路：
+ 把各个组件都需要依赖的同一个状态抽取出来，在全局使用进行管理。
+ 在这种模式下，任何组件都可以直接访问到这个状态，或者当状态发生改变时，所有的组件都获得更新。
## 这时候，就要用到Vuex
这就是 Vuex 背后的基本思想，借鉴了 Flux、Redux。与其他模式不同的是，Vuex 是专门为 Vue 设计的状态管理库，以利用 Vue.js 的数据响应机制来进行高效的状态更新。
## vuex使用周期图
![](vuex.png)
## 那什么时候应该用vuex呢？🤔
+ 如果你不需要开发大型的单页应用，此时你完全没有必要使用vuex，比如你的页面就两三个，使用vuex后增加的文件比你现在的页面还要多，那就没这个必要了。
+ 假如你的项目达到了中大型应用的规模，此时你很可能会考虑，如何更好地在组件外部管理状态，Vuex将会成为你的选择。
# 第二步，安装vuex
1. 首先，安装vuex
```
npm install vuex --save
```
2. 然后配置vuex，在src路径下创建store文件夹，然后创建index.js文件，文件内容如下：
```js
import Vue from 'vue';
import Vuex from 'vuex';
Vue.use(Vuex);
const store = new Vuex.Store({
  state: {
    // 定义一个name，以供全局使用
    name: 'wnxx',
    // 定义一个number，以供全局使用
    number: 0,
    // 定义一个list，以供全局使用
    list: [
      { id: 1, name: 'welcome' },
      { id: 2, name: 'to' },
      { id: 3, name: 'vuex' },
    ],
  },
});
export default store;
```
3. 修改main.js：
```js
import Vue from 'vue';
import App from './App';
import router from './router';
// 引入我们前面导出的store对象
import store from './store'; 
Vue.config.productionTip = false;
new Vue({
  el: '#app',
  router,
  store, // 把store对象添加到vue实例上
  components: { App },
  template: '<App/>',
});
```
4. 最后修改App.vue：
```html
<template>
  <div></div>
</template>
<script>
export default {
  mounted() {
    // 使用this.$store.state.XXX可以直接访问到仓库中的状态
    console.log(this.$store.state.name);
  },
};
</script>
```
此时，启动项目,即可在控制台输出刚才我们定义在store中的name的值。
## 官方建议1: 
官方建议我们以上操作this.$store.state.XXX最好放在计算属性中，就像这样：
```js
export default {
  mounted() {
    console.log(this.getName);
  },
  computed: {
    getName() {
      return this.$store.state.name;
    },
  },
};
```
## 官方建议2:
是不是每次都写this.$store.state.XXX让你感到繁琐，你实在不想写，当然有简便的写法，就像下面这样：
```html
<script>
 // 从vuex中导入mapState
import { mapState } from 'vuex';
export default {
  mounted() {
    console.log(this.name);
  },
  computed: {
    // 经过解构后，自动就添加到了计算属性中，此时就可以直接像访问计算属性一样访问它
    ...mapState(['name']), 
  },
};
</script>
```
你甚至可以在解构的时候给它赋别名，取外号，就像这样：
```js
// 赋别名的话，这里接收对象，而不是数组
...mapState({ aliasName: 'name' }),  
```
🤗现在，安装vuex并且初始化的工作就结束了，此时你可以很轻易的在项目的任意地方访问到仓库里的状态。
# 第三步，了解修饰器：Getter
接下来，我们介绍一个读取操作的 “修饰利器” ---Getter
## 问题假设:
😡设想，你已经将store中的name展示到页面上了，而且是很多页面都展示了，此时产品经理要求：所有的name前面都要加上“hello”！
这时候，你第一想到的是怎么加呢？
emm...在每个页面上，使用this.$store.state.name获取到值之后，进行遍历，前面追加"hello"即可。
No,No,No
## 这样很不好，原因如下：
+ 第一，假如你在A、B、C三个页面都用到了name，那么你要在这A、B、C三个页面都修改一遍，多个页面你就要加很多遍这个方法，这样只会造成代码冗余；
+ 第二，假如下次产品经理让你把 “hello” 改成 “GoodBye” 的时候，你又得把三个页面都改一遍，这时候你只能...😇
## 新的思路:👏🏻 
我们可以直接在store中对name进行一些操作或者加工，从源头解决问题！那么具体应该怎么写呢？
1. 首先，在store对象中增加getters属性
```js
import Vue from 'vue';
import Vuex from 'vuex';
Vue.use(Vuex);
const store = new Vuex.Store({
  state: {
    // 定义一个name，以供全局使用
    name: 'wnxx',
    // 定义一个number，以供全局使用
    number: 0,
    // 定义一个list，以供全局使用
    list: [
      { id: 1, name: 'welcome' },
      { id: 2, name: 'to' },
      { id: 3, name: 'vuex' },
    ],
  },
    // 在store对象中增加getters属性
      getters: {
         getMessage(state) {
      // 获取修饰后的name，第一个参数state为必要参数，必须写在形参上
         return `hello${state.name}`;
    },
  },
});
export default store;
```
2. 在组件中使用:
```js
export default {
  mounted() {
    // 注意不是$store.state了，而是$store.getters
    console.log(this.$store.state.name);
    console.log(this.$store.getters.getMessage);
  },
};
```
## 官方建议:
不要每次都写this.$store.getters.XXX，官方建议我们可以使用mapGetters去解构到计算属性中，就像使用mapState一样，就可以直接使用this调用了，就像下面这样：
```html
<script>
import { mapState, mapGetters } from 'vuex';
export default {
  mounted() {
    console.log(this.name);
    console.log(this.getMessage);
  },
  computed: {
    ...mapState(['name']),
    ...mapGetters(['getMessage']),
  },
};
</script>
```
当然，和mapState一样你也可以取别名，取外号，就像下面这样：
```js
...mapGetters({ aliasName: 'getMessage' }),  
// 赋别名的话，这里接收对象，而不是数组
```
🤗 OK，当你看到这里，你已经成功的了解了getters啦！
😎 读取值的操作我们有 “原生读（state）” 和 “修饰读（getters）”，接下来就要介绍怎么修改值了！
# 第四步，了解如何修改值：Mutation
🤗OK！我们已经成功访问到了store里面的值，接下来我来介绍一下怎么修改state里面的值。
## 错误示范
```js
// 错误示范
this.$store.state.XXX = XXX;
```
## 错误原因:
因为这个store仓库比较奇怪，你可以随便拿，但是你不能随便改。
我举个例子：假如你打开微信朋友圈，看到你的好友发了动态，但是动态里有个错别字，你要怎么办呢？你可以帮他改掉吗？当然不可以！我们只能通知他本人去修改，因为是别人的朋友圈，你是无权操作的，只有他自己才能操作。同理，在vuex中，我们不能直接修改仓库里的值，必须用vuex自带的方法去修改。
## 案例
我们准备完成一个效果：我们先输出state中的number的默认值0，然后我们在vue组件里通过提交Mutations改变number的默认值0，改成我们想修改的值，然后再输出出来。
1. 修改store/index.js
```js
import Vue from 'vue';
import Vuex from 'vuex';
Vue.use(Vuex);
const store = new Vuex.Store({
  state: {
    name: 'wnxx',
    number: 0,
  },
  mutations: {
    // 增加mutations属性
    setNumber(state) {
      // 增加一个mutations的方法，方法的作用是让num从0变成10
      state.number = 10;
    },
  },
});
export default store;
```
2. 修改App.vue
```html
<script>
export default {
  mounted() {
    console.log(`旧值：${this.$store.state.number}`);
    this.$store.commit('setNumber');
    console.log(`新值：${this.$store.state.number}`);
  },
};
</script>
```
以上是简单实现mutations的方法，是没有传参的，如果我们想传不固定的参数怎么办？
1. 修改store/index.js
```js
mutations: {
    setNumber(state) {
      state.number = 10;
    },
    setNumberIsWhat(state, payload) {
      // 增加一个带参数的mutations方法，并且官方建议payload为一个对象
      state.number = payload.number;
    },
  },
```
2. 修改App.vue
```html
<script>
export default {
  mounted() {
    console.log(`旧值：${this.$store.state.number}`);
    // 调用的时候也需要传递一个对象
    this.$store.commit('setNumberIsWhat', { number: 666 }); 
    console.log(`新值：${this.$store.state.number}`);
  },
};
</script>
```
😱这里说一条重要原则：Mutations里面的函数必须是同步操作，不能包含异步操作！
## 官方建议:
就像mapState和mapGetters一样，我们在组件中可以使用mapMutations代替this.$store.commit('XXX')
```html
<script>
import { mapMutations } from 'vuex';
export default {
  mounted() {
    this.setNumberIsWhat({ number: 999 });
  },
  methods: {
    // 注意，mapMutations是解构到methods里面的，而不是计算属性了
    ...mapMutations(['setNumberIsWhat']),
  },
};
</script>
```
当然你也可以给它叫别名，取外号，就像这样：
```js
methods: {
    // 赋别名的话，这里接收对象，而不是数组
  ...mapMutations({ setNumberIsAlias: 'setNumberIsWhat' }), 
},
```
🤪 OK，关于Mutation的介绍大致就是这样。
# 第五步，了解异步操作：Actions
上面提到，Mutations只能进行同步操作，Actions存在的意义是假设你在修改state的时候有异步操作，让你放异步操作。
## 案例
1. 修改store/index.js
```js
const store = new Vuex.Store({
  state: {
    name: 'wnxx',
    number: 0,
  },
  mutations: {
    setNumberIsWhat(state, payload) {
      state.number = payload.number;
    },
  },
  actions: {
    // 增加actions属性
    setNum(content) {
      // 增加setNum方法，默认第一个参数是content，其值是复制的一份store
      return new Promise(resolve => {
        // 我们模拟一个异步操作，1秒后修改number为666
        setTimeout(() => {
          content.commit('setNumberIsWhat', { number: 666 });
          resolve();
        }, 1000);
      });
    },
  },
});
```
2. 修改App.vue
```js
async mounted() {
  console.log(`旧值：${this.$store.state.number}`);
  await this.$store.dispatch('setNum');
  console.log(`新值：${this.$store.state.number}`);
},
```
## 官方建议1:
不要一直使用this.$store.dispatch('XXX')这样的写法调用action，你可以采用mapActions的方式，把相关的actions解构到methods中，用this直接调用：
```html
<script>
import { mapActions } from 'vuex';
export default {
  methods: {
    // 就像这样，解构到methods中
    ...mapActions(['setNum']), 
  },
  async mounted() {
    // 直接这样调用即可
    await this.setNum({ number: 123 }); 
  },
};
</script>
```
当然你也可以给它叫别名，取外号，就像这样：
```js
methods: {
    // 赋别名的话，这里接收对象，而不是数组
  ...mapActions({ setNumAlias: 'setNum' }),
},
```
## 官方建议2:
在store/index.js中的actions里面，方法的形参可以直接将commit解构出来
```js
actions: {
  setNum({ commit }) {
    // 直接将content解构掉，解构出commit，下面就可以直接调用了
    return new Promise(resolve => {
      setTimeout(() => {
        commit('XXXX'); // 直接调用
        resolve();
      }, 1000);
    });
  },
},
```
OK，不要将action和mutation混为一谈，action其实就是mutation的上一级，在action这里处理完异步的一些操作后，后面的修改state就交给mutation去做了。
# 第六步，按属性进行拆分
我们看到，一个store/index.js里面大致包含state/getters/mutations/actions这四个属性，我们可以彻底点，index.js里面就保持这个架子，把里面的内容四散到其他文件中。
我们可以这样拆分：分别是state.js、 getters.js、 mutations.js 、actions.js
1. 拆出来state放到state.js中
```js
// state.js
export const state = {
    name: 'wnxx',
    number: 0,
    list: [
      { id: 1, name: 'welcome' },
      { id: 2, name: 'to' },
      { id: 3, name: 'vuex' },
    ],
  },
```
2. 拆出来getters放到getters.js中
```js
// getters.js
export const getters = {
  getMessage(state) {
    return `hello${state.name}`;
  },
};
```
3. 拆出来mutations放到mutations.js中
```js
// mutations.js
export const mutations = {
  setNumber(state) {
    state.number = 10;
  },
};
```
4. 拆出来actions放到actions.js中
```js
// actions.js
export const actions = {
  setNum(content) {
    return new Promise(resolve => {
      setTimeout(() => {
        content.commit('setNumberIsWhat', { number: 888 });
        resolve();
      }, 1000);
    });
  },
};
```
5. 组装到主文件里面
```js
import Vue from 'vue';
import Vuex from 'vuex';
import { state } from './state'; // 引入 state
import { getters } from './getters'; // 引入 getters
import { mutations } from './mutations'; // 引入 mutations
import { actions } from './actions'; // 引入 actions
Vue.use(Vuex);
const store = new Vuex.Store({ state, getters, mutations, actions });
export default store;
```
# 第七步，按功能进行拆分-Module
上面我们介绍如何拆分项目，采用的是按属性的方式去拆分，接下来，我们介绍一下按另外的一个维度去拆分我们的store，按功能拆分。
1. 我们在之前的store上，增加一个新的仓库store2，主要代码如下：
```js
// store2.js
const store2 = {
  state: {
    name: '我是store2',
  },
  mutations: {},
  getters: {},
  actions: {},
};
export default store2;
```
2. 然后在store中引入我们新创建的store2模块：
```js
import Vue from 'vue';
import Vuex from 'vuex';
import { state } from './state';
import { getters } from './getters';
import { mutations } from './mutations';
import { actions } from './actions';
import store2 from './store2'; // 引入store2模块
Vue.use(Vuex);
const store = new Vuex.Store({
  // 把store2模块挂载到store里面
  modules: { store2 }, 
  state: state,
  getters: getters,
  mutations: mutations,
  actions: actions,
});
export default store;
```
3. 访问state - 我们在App.vue测试访问store2模块中的state中的name，结果如下：
```html
<template>
  <div></div>
</template>
<script>
export default {
  mounted() {
    // 访问store2里面的name属性
    console.log(this.$store.state.store2.name); 
  },
};
</script>
```
我们通过下面的代码可以了解到在不同的属性里是怎么访问模块内的状态或者根状态：
```js
mutations: {
  changeName(state, payload) {
    // state 局部状态
    console.log(state);
    console.log(payload.where);
  },
},
getters: {
  testGetters(state, getters, rootState) {
    // state 局部状态
    console.log(state);
    // 局部 getters,
    console.log(getters);
    // rootState 根节点状态
    console.log(rootState);
  },
},
actions: {
  increment({ state, commit, rootState }) {
    // state 局部状态
    console.log(state);
    // rootState 根节点状态
    console.log(rootState);
  },
},
```
还有一些关于module的内容，比如命名空间、模块注册全局 action、带命名空间的绑定函数、模块动态注册、模块重用等方法，自己查阅即可。🤗
