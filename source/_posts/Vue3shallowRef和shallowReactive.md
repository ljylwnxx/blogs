---
title: Vue3shallowRef和shallowReactive
date: 2022-10-19 19:20:37
tags: vue3
categories: vue3
---
# shallowRef 和 shallowReactive
1. shallowRef 函数，只处理基本类型数据。
2. shallowReactive 函数，只处理第一层数据。
3. 两个在使用的时候都需要引入才可以。
ref 函数和 reactive 函数，他们的作用是将数据转换成响应式的数据，在修改数据的时候，可以将数据实时展示在页面上，基本数据也好，对象也好，都是这样。
但是有一个问题呀，我们在把数据改为响应式数据的时候，不管是用 ref 函数还是使用 reactive 函数，它俩都是深度监听，就是 reactive 包裹的对象，就算有100层，也就是连续点一百个属性那种，去修改最深层的数据，也是可以监听到的，这样的话就会存在问题了。

# 深度监听的问题：
1. 无论 ref 函数还是 reactive 函数都是深度监听。
2. 如果数据量过大，超级超级消耗性能。
3. 如果我们不需要对数据进行深度监听的时候，就可以使用 shallowRef 函数和 shallowReactive 函数。

# 使用 shallowReactive 非深度监听
记住一点，shallowReactive 函数，只能处理第一层数据。
![](shallowReactive一代码.png)
我们分别点击两个按钮，看一下页面变化。
![](shallowReactive一代码结果.png)
通过效果，我们稍微总结一下：
1. shallowReactive只会包装第一层的数据
2. 默认情况它只能够监听数据的第一层。
3. 如果想更改多层的数据，必须先更改第一层的数据，然后在去更改其他层的数据。这样视图上的数据才会发生变化。

# 使用 shallowRef 非深度监听
shallowRef 函数，只能处理基本类型数据，为啥呢，因为 shallowRef 函数监听的是.value 变化。并不是第一层的数据的变化。所以如果要更改 shallowRef 创建的数据，应该 xxx.value = XXX。
![](shallowRef代码.png)
点击按钮，修改 boy 的值
![](shallowRef代码结果.png)

有一个问题：shallowRef 函数，只处理基本类型数据吗？
![](shallowRef二代码.png)
在这个代码里面，我们用 shallowRef 包裹了一个对象， 然后在页面显示 name 和 age ，然后我们通过按钮修改 name 和 age，看一下页面的效果。
![](shallowRef二代码结果.png)
所以说呢，shallowRef 函数，只能处理基本类型数据。
