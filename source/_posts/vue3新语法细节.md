---
title: vue3新语法细节
date: 2023-01-10 18:40:55
tags: vue3
categories: vue3
---
1、在Vue2中，v-for 和 ref 同时使用，这会自动收集 $refs。当存在嵌套的v-for时，这种行为会变得不明确且效率低下。在Vue3中，v-for 和 ref 同时使用，这不再自动收集$refs。我们可以手动封装收集 ref 对象的方法，将其绑定在 ref 属性上。

2、在Vue3中，使用 defineAsyncComponent 可以异步地加载组件。需要注意的是，这种异步组件是不能用在Vue-Router的路由懒加载中。

3、Vue3.0中的 $attrs，包含了父组件传递过来的所有属性，包括 class 和 style 。在Vue2中，$attrs 是接到不到 class 和 style 的。在 setup 组件中，使用 useAttrs() 访问；在非 setup组件中，使用 this.$attrs /setupCtx.attrs 来访问。

4、Vue3中，移除了 $children 属性，要想访问子组件只能使用 ref 来实现了。在Vue2中，我们使用 $children 可以方便地访问到子组件，在组件树中“肆意”穿梭。

5、Vue3中，使用 app.directive() 来定义全局指令，并且定义指令时的钩子函数们也发生了若干变化。

6、data 选项，只支持工厂函数的写法，不再支持对象的写法了。在Vue2中，创建 new Vue({ data }) 时，是可以写成对象语法的。

7、Vue3中新增了 emits 选项。在非<script setup>写法中，使用 emits选项 接收父组件传递过来的自定义，使用 ctx.emit() / this.$emit() 来触发事件。在<script setup>中，使用 defineEmits 来接收自定义事件，使用 defineProps 来接收自定义事件。

8、Vue3中 移除了 $on / $off / $once 这三个事件 API，只保留了 $emit 。

9、Vue3中，移除了全局过滤器（Vue.filter）、移除了局部过滤器 filters选项。取而代之，你可以封装自定义函数或使用 computed 计算属性来处理数据。

10、Vue3 现在正式支持了多根节点的组件，也就是片段，类似 React 中的 Fragment。使用片段的好处是，当我们要在 template 中添加多个节点时，没必要在外层套一个 div 了，套一层 div 这会导致多了一层 DOM结构。可见，片段 可以减少没有必要的 DOM 嵌套。

11、函数式组件的变化：在Vue2中，要使用 functional 选项来支持函数式组件的封装。在Vue3中，函数式组件可以直接用普通函数进行创建。如果你在 vite 环境中安装了 `@vitejs/plugin-vue-jsx` 插件来支持 JSX语法，那么定义函数式组件就更加方便了。

12、Vue2中的Vue构造函数，在Vue3中已经不能再使用了。所以Vue构造函数上的静态方法、静态属性，比如 Vue.use/Vue.mixin/Vue.prototype 等都不能使用了。在Vue3中新增了一套实例方法来代替，比如 app.use()等。

13、在Vue3中，使用 getCurrentInstance 访问内部组件实例，进而可以获取到 app.config 上的全局数据，比如 $route、$router、$store 和自定义数据等。这个 API 只能在 setup 或 生命周期钩子 中调用。

14、我们已经知道，使用 provide 和 inject 这两个组合 API 可以组件树中传递数据。除此之外，我们还可以应用级别的 app.provide() 来注入全局数据。在编写插件时使用 app.provide() 尤其有用，可以替代app.config.globalProperties。

15、在Vue2中，Vue.nextTick() / this.$nextTick 不能支持 Webpack 的 Tree-Shaking 功能的。在 Vue3 中的 nextTick ，考虑到了对 Tree-Shaking 的支持。

