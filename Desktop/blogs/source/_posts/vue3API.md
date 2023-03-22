---
title: vue3API
date: 2022-12-13 21:10:12
tags: vue3
categories: vue3
---
为什么要使用setup组合?
Vue3 中新增的 setup，目的是为了解决 Vue2 中“数据和业务逻辑不分离”的问题。

Vue3中使用 setup 是如何解决这一问题的呢？
第1步: 用setup组合API 替换 vue2 中的data/computed/watch/methods等选项；
第2步: 把setup中相关联的功能封装成一个个可独立可维护的hooks。

1、ref
作用：一般用于定义基本数据类型数据，比如 String / Boolean / Number等。
背后：ref 的背后是使用 reactive 来实现的响应式.
语法：const x = ref(100)
访问：在 setup 中使用 .value 来访问。
![](案例1.png)

2、isRef
作用：判断一个变量是否为一个ref对象。
语法：const bol = isRef(x)
![](案例2.png)

3、unref
作用：用于返回一个值，如果访问的是ref变量，就返回其 .value值；如果不是 ref变量，就直接返回。
语法：const x = unref(y)
![](案例3.png)

4、customRef
作用：自定义ref对象，把ref对象改写成get/set，进一步可以为它们添加 track/trigger。
![](案例4.png)

5、toRef
作用：把一个 reactive对象中的某个属性变成 ref 变量。
语法：const x = toRef(reactive(obj), 'key') // x.value
![](案例5.png)

6、toRefs
作用：把一个reactive响应式对象变成ref变量。
语法：const obj1 = toRefs(reactive(obj))
应用：在子组件中接收父组件传递过来的 props时，使用 toRefs把它变成响应式的。
![](案例6.png)

7、shallowRef
作用：对复杂层级的对象，只将其第一层变成 ref 响应。 (性能优化)
语法：const x = shallowRef({a:{b:{c:1}}, d:2}) 如此a、b、c、d变化都不会自动更新，需要借助 triggerRef 来强制更新。
![](案例7.png)

8、triggerRef
作用：强制更新一个 shallowRef对象的渲染。
语法：triggerRef(shallowRef对象)
参考代码：见shallowRef示例。

9、reactive
作用：定义响应式变量，一般用于定义引用数据类型。如果是基本数据类型，建议使用ref来定义。
语法：const info = reactive([] | {})
![](案例9.png)

10、readonly
作用：把一个对象，变成只读的。
语法：const rs = readonly(ref对象 | reactive对象 | 普通对象)
![](案例10.png)

11、isReadonly
作用: 判断一个变量是不是只读的。
语法：const bol = isReadonly(变量)
![](案例11.png)

12、isReactive
作用：判断一变量是不是 reactive的。
注意：被 readonly代理过的 reactive变量，调用 isReactive 也是返回 true的。
![](案例12.png)

13、isProxy
作用：判断一个变量是不是 readonly 或 reactive的。
![](案例13.png)

14、toRaw
作用：得到返回 reactive变量或 readonly变量的"原始对象"。
语法:：const raw = toRaw(reactive变量或readonly变量)
说明：reactive(obj)、readonly(obj) 和 obj 之间是一种代理关系，并且它们之间是一种浅拷贝的关系。obj 变化，会导致reactive(obj) 同步变化，反之一样。
![](案例14.png)

15、markRaw
作用：把一个普通对象标记成"永久原始"，从此将无法再变成proxy了。
语法：const raw = markRaw({a,b})
![](案例15.png)

16、shallowReactive
作用：定义一个reactive变量，只对它的第一层进行Proxy,，所以只有第一层变化时视图才更新。
语法：const obj = shallowReactive({a:{b:9}})
![](案例16.png)

17、shallowReadonly
作用：定义一个reactive变量，只有第一层是只读的。
语法：const obj = shallowReadonly({a:{b:9}})
![](案例17.png)

18、computed
作用：对响应式变量进行缓存计算。
语法：const c = computed(fn / {get,set})
![](案例18.png)

19、watch
作用：用于监听响应式变量的变化，组件初始化时，它不执行。
语法：const stop = watch(x, (new,old)=>{})，调用stop() 可以停止监听。
语法：const stop = watch([x,y], ([newX,newY],[oldX,oldY])=>{})，调用stop()可以停止监听。
![](案例19.png)

20、watchEffect
作用：相当于是 react中的 useEffect()，用于执行各种副作用。
语法：const stop = watchEffect(fn)，默认其 flush:'pre'，前置执行的副作用。
watchPostEffect，等价于 watchEffect(fn, {flush:'post'})，后置执行的副作用。
watchSyncEffect，等价于 watchEffect(fn, {flush:'sync'})，同步执行的副作用。
特点：watchEffect 会自动收集其内部响应式依赖，当响应式依赖发变化时，这个watchEffect将再次执行，直到你手动 stop() 掉它。
![](案例20.png)

21、生命周期钩子
选项式的 beforeCreate、created，被setup替代了。setup表示组件被创建之前、props被解析之后执行，它是组合式 API 的入口。
选项式的 beforeDestroy、destroyed 被更名为 beforeUnmount、unmounted。
新增了两个选项式的生命周期 renderTracked、renderTriggered，它们只在开发环境有用，常用于调试。
在使用 setup组合时，不建议使用选项式的生命周期，建议使用 on* 系列 hooks生命周期。
![](案例21.png)

22、provide / inject
作用：在组件树中自上而下地传递数据.
语法：provide('key', value)
语法：const value = inject('key', '默认值')
![](案例22.png)

23、getCurrentInstance
作用：用于访问内部组件实例。请不要把它当作在组合式 API 中获取 this 的替代方案来使用。
语法：const app = getCurrentInstance()
场景：常用于访问 app.config.globalProperties 上的全局数据。
![](案例23.png)

24、关于setup代码范式
只使用 setup 及组合API，不要再使用vue选项了。
有必要封装 hooks时，建议把功能封装成hooks，以便于代码的可维护性。
能用 vite就尽量使用vite，能用ts 就尽量使用ts。

