---
title: webpack
date: 2022-08-30 14:25:44
tags: webpack
categories: 知识点
---

# webpack 打包原理是什么？

## 打包原理

webpack 打包原理是根据文件间的依赖关系对其进行静态分析，将这些模块按指定规则生成静态资源，当 webpack 处理程序时，它会递归地构建一个依赖关系图，其中包含应用程序需要的每个模块，将所有这些模块打包成一个或多个 bundle。

# webpack 构建流程：

初始化参数—>开始编译---->确定入口---->编译模块—>完成编译---->输出数据---->输出完成

# 优势：

## 代码层面：

1. 体积更小，加载更快
2. 编译高级语言和语法
3. 兼容性和错误检查

## 研发流程层面：

1. 统一高效的开发环境
2. 统一的构建流程和产出标准
3. 集成公司构建规范（提测、上线）

# 核心概念：

entry：入口，webpack 的执行从 entry 开始，
output：出口，输出结果，webpack 的输出位置，
loader：模块转换器，用于把 webpack 不能直接打包的文件类型转换
plugins：插件，用于把模块原内容按需求转换成新内容
mode：通过选择 development 或者 production 来设置 mode 参数
chunk：代码块，即打包后输出的文件

# 基本功能和工作原理

当源代码没办法直接运行的时候，通过转化将源代码换成可执行的代码，一般包括

1. 代码转换：将无法直接运行的文件代码编译成可以执行的代码
2. 文件优化：压缩文件代码、压缩合并图片等
3. 代码分割：提取多个页面的公共代码、提取首屏不需要执行的部分的代码让其异步加载
4. 模块合并：在有很多模块文件的环境中，需要将模块分类合并成一个文件
5. 自动刷新：监听本地源代码变化，自动重新构建、刷新浏览器
6. 代码校验：在代码被提交前需要校验代码格式等是否符合规范
7. 自动发布：更新完代码后，自动构建出线上发布代码并传输给发布系统

# 常见问题

## 1、loader 和 plugin 的区别：

### 不同的作用：

loader：webpack 将一切文件视为模块，但是 webpack 原生只能解析 JS 文件，如果想打包其他文件的话，就会用到相应的 loader，它是用来让 webpack 拥有加载解析非 JS 文件的工具
plugins：插件，可以扩展 webpack 的功能，让其更具有灵活性

### 不同的用法：

loader：在 module.rules 中配置，也就是说它作为模块的规则存在，类型是数组，数组中的每一项是对象
plugins: 在插件中单独配置，类型是数组，每一项是插件实例，参数都通过构造函数传入

### 2、什么是模块化，为什么要用模块化

模块化是指把一个复杂的系统分解到多个模块以方便编码
不用模块化的话，会出现很多问题，比如无法合理地管理项目依赖跟版本，也无法方便的控制依赖的加载顺序，当项目变大时难以维护，这时候需要模块化来组织代码

### 3、DevServer 开发工具用来自动化（自动编译，自动打开浏览器，自动刷新浏览器）Webpack 在启动时可以开启监听模式，开启监听模式后 Webpack 会监听本地文件系统的变化，发生变化时重新构建出新的结果。

```
devServer: {
    contentBase: resolve(__dirname, 'build'),
    compress: true,
    port: 3000,
    open: true,
    // 开启HMR功能
    // 当修改了webpack配置，新配置要想生效，必须重新webpack服务
    hot: true
  }
```

### 4、什么是 HMR 功能

HMR 又叫热替换，它能在不重新加载整个页面的前提下，通过将更改的模块替换掉被更改的模块，再重新执行实现实时预览
优点：只更新变更内容，以节省宝贵的开发时间。调整样式更加快速，几乎相当于在浏览器中更改样式

### 5、什么是 Tree-sharking?

Tree-sharking 指打包中去除那些引入了但在代码中没用到的死代码（传统的 DCE 方法是除去不可能执行的代码）包括空格、注释等

### 6、babel 和 webpack 的区别

babel： JS 新语法编译工具，只关心语法，不关心模块化
webpack： 打包构建工具，是多个 Loader、 plugin 的集合

### 7、类似 webpack 的工具还有哪些

webpack 适用于大型复杂的前端站点构建
rollup 适用于基础库的打包，如 vue、react
parcel 适用于简单的实验性项目，它可以满足低门槛的快速看到效果由于 parcel 在打包过程中给出的调试信息十分有限，所以一旦打包出错难以调试，所以不建议复杂的项目使用 parcel

### 8、常见 loader

file-loader：把文件输出到一个文件夹中，在代码中通过相对 URL 去引用输出的文件
url-loader：和 file-loader 类似，但是能在文件很小的情况下以 base64 的方式把文件内容注入到代码中去
source-map-loader：加载额外的 Source Map 文件，以方便断点调试
image-loader：加载并且压缩图片文件
babel-loader：把 ES6 转换成 ES5
css-loader：加载 CSS，支持模块化、压缩、文件导入等特性
style-loader：把 CSS 代码注入到 JavaScript 中，通过 DOM 操作去加载 CSS
eslint-loader：通过 ESLint 检查 JavaScript 代码

### 9、常见的 plugins

define-plugin：定义环境变量
commons-chunk-plugin：提取公共代码
uglifyjs-webpack-plugin：通过 UglifyES 压缩 ES6 代码
