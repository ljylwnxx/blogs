---
title: Nuxt3.5正式发布
date: 2023-05-18 20:48:32
tags: Nuxt
categories: vue
---
# 「摘要」
5 月 16 日，Nuxt 3.5.0 正式发布，它带来了 Vue 3.3 版本、新的默认设置、交互式服务端组件、类型化页面、环境配置等。
Nuxt 是使用简便的 Web 框架，用于构建现代和高性能的 Web 应用，可以部署在任何运行 JavaScript 的平台上。去年发布的 Nuxt 3 基于 Vue 3 构建，为 TypeScript 提供了 “一等公民” 支持，并进行了一次彻底的重构，对内核进行了精简，速度更快，体验更好。

# 「一、Vue 3.3」
Vue 3.3 已经发布，具有许多令人兴奋的特性，特别是在类型支持方面。包括：
+ 宏中的导入和复杂类型支持
+ 通用组件
+ 更符合人体工程学的 defineEmits
+ 使用 defineSlots 的类型插槽
+ 响应式 Props 解构
+ defineModel
+ defineOptions
+ 使用 toRef 和 toValue 实现更好的 getter 支持
+ JSX 导入源支持
+ 维护基础设施改进
# 「二、Nitropack v2.4」
Nuxt.js 团队一直致力于对 Nitro 进行大量改进，这些改进已经在 Nitro v2.4 中实现——其中包含许多错误修复、Cloudflare 模块工作格式的更新、Vercel KV 支持等。
注意：如果需要部署到 Vercel 或 Netlify 并希望从增量静态再生中受益，应该更新路由规则：
```
routeRules: {
--  '/blog/**': { swr: 3000 },
++  '/blog/**': { isr: 3000 },
}
```
# 「三、丰富的 JSON 负载」
现在默认启用丰富的 JSON 负载序列化。这既更快又允许序列化从 Nuxt 服务端传递到客户端的有效载荷中的复杂对象（以及在为预渲染站点提取有效载荷数据时）。
现在这意味着开箱即用地支持各种丰富的 JS 类型：正则表达式、日期、Map 和 Set 以及 BigInt 以及 NuxtError，以及 Vue 特定的对象，如 ref、reactive、shallowRef 和 shallowReactive。
长期以来，由于序列化 Errors 和其他非 POJO 对象的问题，Nuxt 一直在使用自己的 devalue fork，但现在已经过渡回原始版本。我们甚至可以使用新的对象语法Nuxt插件注册自定义类型：
```
export default definePayloadPlugin(() => {
  definePayloadReducer('BlinkingText', data => data === '<original-blink>' && '_')
  definePayloadReviver('BlinkingText', () => '<revivified-blink>')
})
```
# 「四、交互式服务端组件」
这个功能应该被认为是高度实验性的，现在通过插槽支持服务端组件内的交互式内容。

# 「五、环境配置」
现在可以在 nuxt.config 中配置完全类型化的、按环境的覆盖：
```
export default defineNuxtConfig({
  $production: {
    routeRules: {
      '/**': { isr: true }
    }
  },
  $development: {
    //
  }
})
```
如果正在创作图层，还可以使用 $meta 关键字来提供元数据，层的使用者可能会用到它。
# 「六、完全类型化的页面」
通过与unplugin-vue-router的实验性整合，可以在Nuxt应用中受益于完全类型化的路由。
开箱即用，这将启用对navigateTo、<NuxtLink>、router.push()等功能的类型化使用。
甚至可以通过使用const route = useRoute('route-name')在页面内获取类型化的参数。
直接在nuxt.config中启用此功能：
```
export default defineNuxtConfig({
  experimental: {
    typedPages: true
  }
})
```
# 「七、Bundler的模块解析」
现在，Nuxt内部完全支持使用 bundler 策略进行模块解析。
如果可能，建议采用此方法。它具有对子路径导出的类型支持，但与 Node16 解析相比，更准确地匹配了诸如 Vite 和 Nuxt 这样的构建工具的行为。
```
export default defineNuxtConfig({
  typescript: {
    tsConfig: {
      compilerOptions: {
        moduleResolution: 'bundler'
      }
    }
  }
})
```
这将开启TypeScript跟踪Node子路径导出的功能。例如，如果一个库有一个像mylib/path这样的子路径导出，映射到mylib/dist/path.mjs，那么可以从mylib/dist/path.d.ts中引入此类型，而不需要库作者创建mylib/path.d.ts。
# 「八、分离的服务端类型」
Nuxt 团队计划通过为~/server目录生成单独的tsconfig.json来改善IDE中"nitro"和"vue"部分之间的清晰度。
可以通过添加一个额外的~/server/tsconfig.json并使用以下内容来使用：
```
{
  "extends": "../.nuxt/tsconfig.server.json"
}
```
虽然现在在类型检查 (nuxi typecheck) 时不会考虑这些值，但应该在 IDE 中获得更好的类型提示。
# 「九、弃用内容」
虽然没有为 Nuxt 2 中的build.extend钩子提供类型或文档，但一直在webpack 中调用它，现在明确地弃用了它，并将在将来的小版本中删除它。
# 「十、升级」
建议运行以下命令进行升级：
```
nuxi upgrade --force
```
这也将刷新 lockfile 文件，并确保从 Nuxt 依赖的其他依赖项中获取更新，特别是在 unjs 生态系统中。
参考：https://nuxt.com/blog/v3-5

