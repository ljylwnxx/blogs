---
title: 打开vite大门
date: 2023-04-24 09:49:16
tags: vite
categories: vite
---
# 前言
关于前端构建工具，先有 gulp、grunt、webpack，现在尤大大又整了一个 vite；
或许有同学会疑问，这么多构建工具为什么只有 vite 快，那么我们为什么要使用 vite，以及到底该选哪一个呢？
# Vite 简介
## 什么是 Vite
Vite是一个由原生ES Module驱动的Web开发前端构建工具。
在开发环境（Development） 下基于浏览器原生 ES Module 开发，完全跳过了打包这个概念；
在生产环境（Production） 下基于 Rollup 打包来构建代码。
## Vite 的主要特性
+ 💡 极速的服务启动： 使用原生 ESM 文件，无需打包！
+ ⚡️ 轻量快速的热重载： 无论应用程序大小如何，都始终极快的模块热重载（HMR）
+ 🛠️ 丰富的功能： 对 TypeScript、JSX、CSS 等支持开箱即用。
+ 📦 优化的构建：可选 “多页应用” 或 “库” 模式的预配置 Rollup 构建
+ 🔩 通用的插件： 在开发和构建之间共享 Rollup-superset 插件接口。
+ 🔑 完全类型化的API：灵活的 API 和完整 TypeScript 类型
# Vite 的优势
+ 上手非常简单
+ 开发效率极高
+ 社区成本低（兼容绝大部分 rollup 插件）
## 开发效率极高 —— 🚀 般的速度
![](vite速度.png)
从图中对比我们可以很明显的发现，为什么 Vite 会较于 Webpack 快那么多了
Vite 在开发环境冷启动无需打包，无需分析模块之间的依赖，同时也无需在启动开发服务器前进行编译，启动时还会使用 esbuild 来进行预构建；（如下图基于原生 ESM 的开发服务流程图）
![](esm.png)
Webpack 在启动后会做一堆事情，经历一条很长的编译打包链条，从入口开始需要逐步经历语法解析、依赖收集、代码转译、打包合并、代码优化，最终将高版本的、离散的源码编译打包成低版本、高兼容性的产物代码，这可满满都是 CPU、IO 操作啊，在 Node 运行时下性能必然是有问题。（如下图基于 bundle 的开发服务流程图）
![](bundle.png)
当然我们只做了与前端使用率最高，名气最大的 webpack 的对比，像其它构建工具如 Browserify、Gulp、Parcel、Rollup、Snowpack 几乎都是可以类似对比处理。
## 上手极其简单
命令初始化
```
Vite2.x 需要 Node.js 版本 >= 12.0.0
Vite3.x 不再支持 Node.js 12，现在需要 Node.js 14.18+
```
简单一句话概括，如果说你会使用 vue-cli 脚手架，那么这个 Vite 构建工具你几乎可以无缝插入；
```
 # npm
 npm init vite@latest
 ​
 # yarn
 yarn create vite
 ​
 # pnpm
 pnpm create vite
```
然后按照提示选择你所需要创建的项目即可。
配置极其简单
相比 webpack（或者底层依赖 Webpack 的 vue-cli）， 需要对 entry、loader、plugin 等进行诸多配置，实际的构建工作通常由各种 webpack loader、plugin 实现；
Vite 的使用可谓是相当简单了；只需执行初始化命令，就可以得到一个预设好的开发环境，开箱即获得一堆功能，包括：css 预处理、html 预处理、异步加载、分包、压缩、HMR支持、默认 chunk hash 命名 等。
简单说吧，Vite 的定位就是傻瓜且强大的构建工具。
## 社区成本低
除了极致的运行性能与简易的使用方法外，Vite 对已有生态的兼容性也不容忽略，主要体现在
两个点：
+ 与 Vue 解耦，兼容支持 React、Svelte、Preact、Vanilla 等，这意味着 Vite 可以被应用在大多数现代技术栈中
+ 与 Rollup 极其接近的插件接口，这意味着可以复用 Rollup 生态中大部分已经被反复锤炼的工具
讲真的，这两条摆上桌面，加上前面讨论的 性能优势 和 超低学习成本，你还有什么拒绝的理由呢？
# Vite 4.3 正式发布，构建速度全面提升
最新发布的 Vite 4.3 显著提升了性能。发布公告写道，Vite 团队在这个版本中将工作重心放在提升开发服务器的性能上，其中包括简化解析逻辑、改进热路径、实现更智能的缓存以查找 package.json,TS 配置文件和解析的 URL。  
# 为什么这么快
## 更智能的解析策略
Vite会将所有接收到的URL和路径解析为目标模块。在 Vite 4.2 中，存在很多冗余的解析逻辑和不必要的模块搜索。为了减少计算和文件系统调用，Vite 4.3 使解析逻辑更简单、更严格和更准确。
## 更简单的解析
Vite 4.2严重依赖 resolve 包来解析依赖的 package.json，查看 resolve 的源码发现解析 package.json 时有很多无用的逻辑。Vite 4.3 摒弃了 resolve，遵循更简单的 resolve 逻辑：直接检查嵌套父目录中是否存在 package.json。
## 更严格的解析
Vite 必须调用 Nodejs fs API 来查找模块。但是 IO 很昂贵。Vite 4.3 缩小了文件搜索范围，并跳过搜索一些特殊路径，以尽可能减少 fs 调用。例如：
+ 由于 # 符号不会出现在 URL 中，用户可以控制源文件路径中没有 # 符号，因此 Vite 4.3 不再检查用户源文件中带有 # 符号的路径，而是仅在 node_modules 中搜索它们。
+ 在Unix系统中，Vite 4.2 会先检查根目录下的每一个绝对路径，对大多数路径都可以，但是如果绝对路径以根开头就很容易失败。为了在 /root/root 不存在的情况下跳过搜索 /root/root/path-to-file，Vite 4.3 会在开头判断 /root/root 作为目录是否存在，并预先缓存结果。
+ 当 Vite 服务器收到 @fs/xxx​ 和 @vite/xxx 时，就不需要再解析这些 URL。Vite 4.3 直接返回之前缓存的结果，不再重新解析。
## 更准确的解析
Vite 4.2 在文件路径为目录时递归解析模块，会导致不必要的重复计算。Vite 4.3 将递归解析扁平化，并对不同类型的路径应用适当的解析，展平后缓存一些 fs 调用也更容易。
## 包解析
Vite 4.3 打破了解析 node_modules 包数据的性能瓶颈。Vite 4.2 使用绝对文件路径作为包数据缓存键。这还不够，因为 Vite 必须遍历 pkg/foo/bar​ 和 pkg/foo/baz 中的同一个目录。
Vite 4.3 不仅使用了绝对路径(/root/node_modules/pkg/foo/bar.js​ & /root/node_modules/pkg/foo/baz.js​)，还使用了遍历目录(/root/node_modules/pkg/foo​ & /root/node_modules/pkg​) 作为 pkg 缓存的键。
另一种情况是，Vite 4.2 在单个函数中查找深层导入路径的 package.json​，例如 Vite 4.2 解析 a/b/c/d​ 等文件路径时，首先检查根 a/package.json​ 是否存在， 如果没有，则按照a/b/c/package.json​ -> a/b/package.json​的顺序查找最近的package.json​，但事实是查找根package.json​和最近的package.json​应该分开处理 ，因为在不同的解析上下文中需要它们。Vite 4.3 将根 package.json​ 和最近的 package.json 解析分成两部分，这样它们就不会混在一起。 
## fs.realpathSync
Nodejs 中有一个有趣的 realpathSync 问题，它指出 fs.realpathSync​ 比 fs.realpathSync.native​ 慢 70 倍。但 Vite 4.2 仅在非 Windows 系统上使用 fs.realpathSync.native​，因为它在 Windows 上的行为不同。为了解决这个问题，Vite 4.3 在 Windows 上调用 fs.realpathSync.native 时添加了网络驱动器验证。
## 非阻塞任务
作为一个按需服务，Vite dev server 可以在没有准备好所有东西的情况下启动。
## 非阻塞 tsconfig 解析
Vite 服务器在预绑定 ts 或 tsx 时需要 tsconfig 数据。
Vite 4.2 在服务端启动之前，在插件钩子 configResolved​ 中等待 tsconfig​ 数据解析完成。一旦服务器启动而没有准备好 tsconfig​ 数据，页面请求就可以访问服务器，即使请求可能需要稍后等待 tsconfig 解析。
Vite 4.3 会在服务器启动前初始化 tsconfig​ 解析，但服务器不会等待。解析过程在后台运行。一旦有ts相关的请求进来，就得等tsconfig解析完了。
## 非阻塞文件处理
Vite 中有大量的 fs 调用，其中一些是同步的。这些同步 fs 调用可能会阻塞主线程。Vite 4.3 将它们改为异步。此外，并行化异步函数也更容易。关于异步函数，可能有许多 Promise 对象在解析后要释放。由于更智能的解析策略，释放 fs-Promise 对象的成本要低得多。
## HMR 防抖
考虑两个简单的依赖链 C <- B <- A & D <- B <- A，当 A 被编辑时，HMR 将从 A 传播到 C 和 A 传播到 D。这导致 A 和 B 在 Vite 4.2 中被更新两次。
Vite 4.3 缓存了这些遍历的模块，以避免多次搜索它们，它适用于由 git checkout 触发的 HMR。
## 并行化
并行化始终是获得更好性能的好选择。在 Vite 4.3 中，我们并行化了一些核心功能，包括导入分析、提取 deps 的导出、解析模块 url 和运行批量优化器。
## Javascript 优化
## 用回调替换 *yield
Vite 使用 tsconfck​ 来查找和解析 tsconfig​ 文件。tsconfck​ 曾经通过 yield​ 遍历目标目录，生成器的一个缺点是它需要更多的内存空间来存储它的生成器对象，并且在运行时会有大量的生成器上下文切换。所以从 v2.1.1 开始在核心中用回调替换 ​​yield​​。
3.2 使用 === 代替 startsWith 和 endsWith
Vite 4.2 使用 startsWith​ 和 endsWith​ 检查热更新 URL 中的头部和尾部 '/'。比较 str.startsWith('x')​ 和 str[0] === 'x'​ 的执行基准发现，===​ 比 startsWith​ 快约 20%，endsWith 比 === 慢约 60%。
## 避免重复创建正则表达式
Vite 需要很多正则表达式来匹配字符串，其中大部分都是静态的，因此只使用它们的单例会更好。Vite 4.3 将正则表达式提升，以便可以重复使用它们。
## 放弃生成自定义错误
在 Vite 4.2 中，有一些自定义错误以改进开发体验。这些错误可能会导致额外的计算和垃圾回收，从而降低 Vite 的速度。在 Vite 4.3 中，放弃了生成某些热更新自定义错误（例如 package.json NOT_FOUND 错误），直接抛出原始错误以获得更好的性能。