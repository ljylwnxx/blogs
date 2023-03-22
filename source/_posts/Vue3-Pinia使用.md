---
title: 'Vue3:Pinia使用'
date: 2022-10-24 21:44:07
tags: vue3
categories: vue3
---
在 vue2 版本中，如果想要使用状态管理器，那么一定是集成 Vuex，首先说明一点，Vuex 在 vue3 项目中依旧是可以正常使用的，是 vue 项目的正规军。但是，今天我们学习一下Pinia 目前也已经是 vue 官方正式的状态库。适用于 vue2 和 vue3。可以简单的理解成 Pinia 就是 Vuex5。也就是说， Vue3 项目，建议使用Pinia，当然很多公司或者是项目由 vue2 转为 vue3 之后，由于习惯了使用 vuex ，所以说，在 vue3 当中继续使用 vuex 的，也不是少数，都知道就可以，根据实际情况来选择。

# 什么是 Pinia
Pinia 是 Vue 的存储库，它允许您跨组件/页面共享状态。Pinia 的成功可以归功于他管理存储数据的独特功能，例如：可扩展性、存储模块组织、状态变化分组、多存储创建等。
# Pinia 的优点
Pinia 被 vue 纳入正规编制，肯定是有原因的，那 pinia 有啥优点呢，主要是一下几点：
pinia 符合直觉，易于学习。
pinia 是轻量级状态管理工具，大小只有1KB.
pinia 模块化设计，方便拆分。
pinia 没有 mutations，直接在 actions 中操作 state，通过 this.xxx 访问响应的状态，尽管可以直接操作 state，但是还是推荐在 actions 中操作，保证状态不被意外的改变。
store 的 action 被调度为常规的函数调用，而不是使用 dispatch 方法或者是 MapAction 辅助函数，这是在 Vuex 中很常见的。
支持多个 store。
支持 Vue devtools、SSR、webpack 代码拆分。

# 相关资料
Pinia 中文网：https://pinia.web3doc.top/

# Pinia 安装
安装 pinia 就很简单了，直接命令安装就可以了。
![](安装.png)

# Pinia 使用
安装完 pinia ，然后就是使用了，使用的第一步，就是在项目中引入 pinia
## 1.Pinia 导入
首先在 main.js 文件中引入
 vue3 的写法：
import {createPinia} from 'pinia'
然后，这个 pinia 就在项目中导入了
Pinia 是支持 vue2 的，如果是 vue2 的项目，导入的方式是下面的样子：
import {PiniaVuePlugin} from 'pinia'
我们还是以 vue3 来介绍这个 Pinia
导入的时候是 hook ，我们需要调用一下
const state = createPinia()
调用完成，state 是以插件的形式存在的，所以说最后我们需要在项目使用一下。
app.use(state)
## 2.Pinia 基本使用
### 1.创建 index.ts 文件
使用起来相对简单一些，我们首先在根目录下创建一个 store 文件夹，用 vuex 的时候也是这个结构。毕竟 pinia 就是用来替换掉 vuex 的嘛。
创建完 store 文件夹，在里面创建一个 ts 文件，叫做 index.ts 。
### 2.编写 index.ts 文件
首先我们先引入 pinia
import { defineStore } from "pinia";
由于 defineStore 也是一个 hooks ，所以说我们可以直接导出一下
这样子写是会报错的，因为这个 defineStore 是需要传参数的，其中第一个参数是id，就是一个唯一的值，简单点说就可以理解成是一个命名空间，我们可以写一个枚举再传值。
我们在同级在创建一个名字叫做 store_name.ts 的文件写一个枚举数据导出
export const enum Names {
  TEST = "TEST"
}
***
然后在 index.ts 文件中引入一下枚举数据，然后传给这个 defineStore。
 这个样子还是报错的，因为还有其他的参数需要传递，第二个参数就是一个对象，里面有三个模块需要处理，第一个是 state，第二个是 getters，第三个是 actions。
state 和之前我们 vuex 里面的写法是不一样的，在 vuex 里面呢，state 是一个对象，但是在 pinia 中，state 必须是一个箭头函数，然后在返回一个对象。
getters 模块呢，类似于计算属性，是有缓存的，主要是帮助我们修饰一些值。
actions 呢，类似于 methods，帮助我们做一些同步的或者是异步的操作，提交 state 之类的。
![](indexts.png)
其实到这个地方为止呢，其实我们已经可以使用 pinia 了，我们写一个页面使用一下 pinia 的值。
![](截图1代码.png)
![](截图1结果.png)
### 3.修改 Pinia 的值
修改 Pinia 的值有很多中方式，修改 pinia 的值，其实就是修改 state 的值
修改值的方式呢，常见的有五种
#### 方式一：直接修改
![](截图2代码.png)
![](截图2结果.png)
#### 方式二：$patch 函数修改
在我们实例化 const userInfo = useInfoStore() 这个 state 的时候，其实这个 userInfo 中，有一个方法，就是 patch 函数，它可以帮助我们批量修改。
![](截图3代码.png)
![](截图3结果1.png)
![](截图3结果2.png)
#### 方式三：$patch 函数修改
咦，为什么和第二种一样，都是使用 patch 函数来实现修改，是有区别，方式二是在 patch 函数中传入修改的对象值，但是这种方式传入的是一个函数，作用是啥子呢？就是可以进行逻辑操作，比如说判断之类的。
![](截图4代码.png)
效果其实和方式二是一模一样的
#### 方式四：$state 方式
![](截图5代码.png)
#### 方式五： action 方式
这个方式我们需要借助 actions 来实现，所以说我们需要去 store 文件夹下的 index.ts 文件中写一个 action
![](截图6代码1.png)
写完 action 我们就可以使用 action 的方式修改 age 的值了。怎么使用呢，我们只需要使用实例去调用一下我们刚才写的 action 函数就行了。
![](截图6代码2.png)
查看一下效果，可以看到
![](截图6结果1.png)
![](截图6结果2.png)
当然了，这个 action 是可以传参的，我们修改一下 index.ts 文件编写的 action 函数，让他接受一个参数再赋值。
![](截图7代码1.png)
![](截图7代码2.png)
查看一下效果，可以看到
![](截图7结果1.png)
![](截图7结果2.png)
以上就是常见的五种修改 state 数据的方式，可以根据自己的实际情况选择使用
# pinia 解构
上面的案例，我们实例化const userInfo = useInfoStore()了以后呢，这个 userInfo 是可以继续解构操作的
![](截图8代码.png)
在页面分别展示一下解构前和解构后的数据，查看一下效果，可以看到
![](截图8结果.png)
我们可以看到，结构前和结构后我们在页面都可以获取到数据展示。
但是有个问题，就是解构后的数据是不具备响应式的，意思就是我们修改了 state 的值，页面不会跟着变化。
做个测试，点击按钮，age 加一，看一下结构前和解构后的数据在页面是否会实时渲染
![](截图9代码.png)
在页面分别展示一下解构前和解构后的数据，查看一下效果，可以看到
![](截图9代码.png)
通过测试我们可以看到，结构前的是可以实时渲染的，但是解构后的话是不可以的， 因为解构后的不是响应式数据。
解决这个问题很简单，官方提供了一个方法，可以把解构后的数据转换为响应式的数据。
就是 storeToRefs，使用 storeToRefs 需要导入一下。
import { storeToRefs } from 'pinia'
然后把我们解构的对象包裹一下就可以了
const { name, age } = storeToRefs(userInfo)
![](截图10代码.png)
![](截图10结果.png)
或者我们换一个写法，直接操作结构后的数据，记得，要 .value
![](截图11代码.png)
# Pinia 的 actions
actions 是可以处理同步，也可以处理异步，同步的话相对来说简单一点，上面我们通过 action 修改 state 的时候，就用到了 actions 的同步
接下来我们重点介绍一下actions异步
actions 异步
首先我们模拟一个异步函数，然后我们在 actions 中可以调用这个异步操作，在 actions 处理异步的时候，我们一般是与 async 和 await 连用
![](截图11代码1.png)
![](截图11代码2.png)
查看一下效果，可以看到
![](截图11结果1.png)
![](截图11结果2.png)
actions 同步、异步连用
这个 actions 里面的方法函数是可以相互调用的，意思就是你 actions 里面有好几个方法，这几个方法是可以调过来调过去的。
上面的代码改一下，本来异步模拟获取的 age 数据是 8 ，然后我们调用一个 action 把 age 改成 18。
![](截图12代码.png)
查看一下效果，可以看到
![](截图12结果.png)
# getter 函数
getters 类似于 vue 里面的计算属性，可以对已有的数据进行修饰。
有两种写法:
普通函数方式写法
![](截图13代码1.png)
![](截图13代码2.png)
查看一下效果，可以看到
![](截图13结果1.png)
然后我们点一下按钮，修改一下 name，然后看一下效果
![](截图13结果2.png)
我们可以看见，点击修改 name 之后getter 也会实时的渲染出来
相互调用
![](截图14代码.png)
查看一下效果，可以看到
![](截图14结果1.png)
然后我们点一下按钮，修改一下 name和 age，然后看一下效果
![](截图14结果2.png)
# API 的使用
$reset ：重置到初始值
这个 $reset 可以将 state 的数据重置到初始值，比如我们有一个数据，点击按钮改变了，然后我们可以通过这个 API ，将数据恢复到初始状态值。
![](截图15代码.png)
我们点击修改用户按钮，查看一下效果，可以看到
![](截图15结果1.png)
然后我们点一下$reset按钮，然后看一下效果
![](截图15结果2.png)
$subscribe：监听 state 数据变化
$subscribe 使用来监听的，监听 state 数据的变化，只要 state 里面的数据发生了变化，就会自动走这个函数。
![](截图16代码.png)
我们点击修改用户按钮，查看一下效果，可以看到
![](截图16结果1.png)
控制台打印出监听到的变化结果
![](截图16结果2.png)
$onAction：一调用 actions 就触发
![](截图17代码.png)
控制台打印出监听到的变化结果
![](截图17结果.png)