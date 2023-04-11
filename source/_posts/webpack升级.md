---
title: webpack升级
date: 2023-04-05 16:15:20
tags: webpack
categories: webpack
cover: [/images/webpackcover.png]
banner: 
  type: img
  bgurl: [/images/webpackcover.png]
---
# 一、背景
由于项目越来越庞大复杂，打包时间也非常长，本地开发环境每次重启都要打包好久，正好借此契机对webpack做了一个升级。
# 二、webpack5 和 webpack4 的区别有哪些 ？
## Tree Shaking
### 作用：
如果我们的项目中引入了lodash包，但是我只用了其中的一个方法。其他没有用到的方法是不是冗余的？此时tree-shaking就可以把没有用的那些东西剔除掉，来减少最终的bundle体积。
usedExports : true, 标记没有用的叶子
minimize: true, 摇掉那些没有用的叶子
```
// webpack.config.js中
  module.exports = {
     optimization: {
       usedExports: true, //只导出被使用的模块
       minimize : true // 启动压缩
     }
  }
```
由于tree shaking只支持esmodule ，如果你打包出来的是commonjs，此时tree-shaking就失效了。不过当前大家都用的是vue，react等框架，他们都是用babel-loader编译，以下配置就能够保证他一定是esmodule。
![](webpack4.png)
webpack5的 mode=“production” 自动开启 tree-shaking。
## 压缩代码
+ webpack4
webpack4需要下载安装terser-webpack-plugin 插件，并且需要以下配置
```
const TerserPlugin = require('terser-webpack-plugin')
module.exports = { 
// ...other config
optimization: {
  minimize: !isDev,
  minimizer: [
    new TerserPlugin({
      extractComments: false, 
      terserOptions: { 
        compress: { 
          pure_funcs: ['console.log'] 
        }
      }
    }) 
   ]
 }
}
```
+ webpack5
内部本身就自带 js 压缩功能，他内置了 terser-webpack-plugin 插件，我们不用再下载安装。而且在 mode=“production” 的时候会自动开启 js 压缩功能。
如果你要在开发环境使用，就用下面：
```
// webpack.config.js中
  module.exports = {
     optimization: {
       usedExports: true, //只导出被使用的模块
       minimize : true // 启动压缩
     }
  }
```
+ js 压缩失效问题
当你下载 optimize-css-assets-webpack-plugin ，执行 css 压缩以后，你会发现 webpack5 默认的 js 压缩功能失效了。先说 optimize-css-assets-webpack-plugin 的配置：
```
npm install optimize-css-assets-webpack-plugin -D
```
```
module.exports = { 
  optimization: { 
    minimizer: [ 
      new OptimizeCssAssetsPlugin() 
    ]
  },
}
```
此时的压缩插件optimize-css-assets-webpack-plugin可以配置到plugins里面去，也可以如图配置到到 optimization里面。区别如下：
配置到plugins中，那么这个插件在任何情况下都会工作。而配置在optimization表示只有minimize为 true 的时候才能工作。
当安装 optimize-css-assets-webpack-plugin 以后你去打包会发现原来可以压缩的 js 文件，现在不能压缩了。原因是你指定的压缩器是optimize-css-assets-webpack-plugin
导致默认的terser-webpack-plugin就会失效。解决办法如下：
```
npm install terser-webpack-plugin -D
```
```
optimization: {
    minimizer: [
      new TerserPlugin({
        extractComments: false,
        terserOptions: {
          compress: { pure_funcs: ['console.log'] },
        },
      }),
      new OptimiazeCssAssetPlugin(),
    ]
  }
```
即便在webpack5中，你也要像webpack4中一样使用js压缩。
+ 注意事项
在webpack5里面使用optimize-css-assets-webpack-plugin会报错，因为官方已经打算要废除了，请使用替换方案：
```
npm i css-assets-webpack-plugin -D
```
## 合并模块
普通打包只是将一个模块最终放到一个单独的立即执行函数中，如果你有很多模块，那么就有很多立即执行函数。concatenateModules可以要所有的模块都合并到一个函数里面去。
optimization.concatenateModules = true
配置如下：
```
module.exports = {
  optimization: {
    usedExports: true,
    concatenateModules: true,
    minimize: true
  }
}
```
此时配合 tree-shaking 你会发现打包的体积会减小很多。
## 副作用sideEffects
+ webpack4 新增了一个sideEffects的功能，允许我们通过配置来标识我们的代码是否有副作用。这个特性只有在开发npm包的时候用到。
+ 副作用的解释： 在utils文件夹下面有index.js文件，用于系统导出utils里面其他文件，作用就是写的少，不管utils里面有多少方法，我都只需要引入utils即可。
```
// utils/index.js
  export * from './getXXX.js';
  export * from './getAAA.js';
  export * from './getBBB.js';
  export * from './getCCC.js';
```
```
 // 在其他文件使用 getXXX 引入
  import {getXX} from '../utils'
```
+ 此时，如果文件getAAA在外界没有用到，而tree-shaking又不能把它摇掉怎么办？
+ 这个getAAA就是副作用。你或许要问tree-shaking为什么不能把它摇掉？
+ 原因就是：他在utils/index.js 里面使用了。只能开启副作用特性。如下：
```
// package.json中
{
  name：“项目名称”,
  ....
  sideEffects: false
}
```
```
// webpack.config.js
module.exports = {
  mode: 'none',
  ....
  optimization: {
    sideEffects: true
  }
}
```
+ 副作用开启：
(1)optimization.sideEffects = true 开启副作用功能
(2)package.json 中设置 sideEffects : false 标记所有模块无副作用
+ 说明： 
webpack打包前都会检查项目所属的package.json文件中的sideEffects标识，如果没有副作用，那些没有用到的模块就不需要打包，反之亦然。此时，在webpack.config.js里面开启sideEffects。
## webpack 缓存
+ webpack4缓存配置
支持缓存在内存中
```
npm install hard-source-webpack-plugin -D
```
```
const HardSourceWebpackPlugin = require('hard-source-webpack-plugin') 
module.exports = { 
plugins: [
  // 其它 plugin... 
  new HardSourceWebpackPlugin(), 
] }
```
+ webpack5缓存配置
webpack5内部内置了cache缓存机制，直接配置即可。
cache会在开发模式下被设置成type：memory而且会在生产模式把cache给禁用掉。
```
// webpack.config.js
module.exports= {
  // 使用持久化缓存
  cache: {
    type: 'filesystem'，
    cacheDirectory: path.join(__dirname, 'node_modules/.cac/webpack')
  }
}
```
+ type的可选值为：memory使用内容缓存，filesystem使用文件缓存。
+ 当type=filesystem的时候设置cacheDirectory才生效。用于设置你需要的东西缓存放在哪里。
## 对loader的优化
webpack4加载资源需要用不同的loader
+ raw-loader将文件导入为字符串
+ url-loader将文件作为data url内联到bundle文件中
+ file-loader将文件发送到输出目录中
![](webpack4-1.png)
webpack5 的资源模块类型替换 loader
资源模块类型(asset module type)，通过添加4种新的模块类型，来替换所有这些 loader：
+ asset/resource 发送一个单独的文件并导出 URL。之前通过使用 file-loader 实现。
+ asset/inline 导出一个资源的 data URI。之前通过使用 url-loader 实现。
+ asset/source 导出资源的源代码。之前通过使用 raw-loader 实现。
+ asset 在导出一个data URI和发送一个单独的文件之间自动选择。之前通过使用 url-loader，并且配置资源体积限制实现。
![](webpack4-2.png)
## 启动服务的差别
+ webpack4 启动服务
通过webpack-dev-server启动服务
+ webpack5 启动服务
内置使用webpack serve启动，但是它的日志不是很好，所以一般都加都喜欢用webpack-dev-server优化。
## devtool的差别
sourceMap需要在webpack.config.js里面直接配置devtool就可以实现了。而 devtool有很多个选项值，不同的选项值，不同的选项产生的 .map 文件不同，打包速度不同。
一般情况下，我们一般在开发环境配置用“cheap-eval-module-source-map”，在生产环境用‘none’。
devtool在webpack4和webpack5上也是有区别的
v4: devtool: 'cheap-eval-module-source-map'
v5: devtool: 'eval-cheap-module-source-map'
## 热更新差别
+ webpack4设置
![](webpack4-3.png)
+ webpack5设置
如果你使用的是bable6，按照上述设置，你会发现热更新无效，需要添加配置：
```
 module.hot.accept('需要热启动的文件',(source)=>{
     //自定义热启动
  })
```
当前最新版的babel里面的 babel-loader已经帮我们处理的热更新失效的问题。所以不必担心，直接使用即可。
如果你引入 mini-css-extract-plugin 以后你会发现 样式的热更新也会失效。
只能在开发环境使用style-loader，而在生产环境用MinicssExtractPlugin.loader。 如下：
![](webpack5-1.png)
## 使用 webpack-merge 的差别
+ webpack4 导入
```
const merge = require('webpack-merge);
```
+ webpack 5 导入
```
const {merge} = require('webpack-merge');
```
## 使用 copy-webpack-plugin 的差别
```
//webpack.config.js
const CopyWebpackPlugin = require('copy-webpack-plugin');
module.exports = {
  plugins: [
    // webpack 4
    new CopyWebpackPlugin(['public']),   
    // webpack 5
    new CopyWebpackPlugin({
      patterns: [{
        from: './public',
        to: './dist/public'
      }]
    })
  ]
}
```
webpack5支持的新版本里面需要配置的更加清楚。
# 三、升级过程
Webpack5对Node.js的版本要求至少是10.13.0
## 先升级 webpack 和 webpack-cli
```scss
npm install --save-dev webpack@latest webpack-cli@latest  webpack-dev-server@latest webpack-merge@latest
```
webpack-merge升级以后，使用方式改为如下：
```js
const webpackMerge = require("webpack-merge");
```
```js
const { merge } = require('webpack-merge');
```
升级所有使用到的plugin和loader为最新的可用版本。
部分plugin和loader可能会有一个beta版本，必须使用它们才能与webpack 5兼容。
## 执行npm start
在 package.json中scripts的start命令如下：
```json
"scripts": {
    "start": "cross-env NODE_ENV=dev webpack-dev-server --hot --progress --colors  --config ./webpack.dev.js",
}
```
### --colors 报错
在v4版本中，我们可以使用 --colors或者 --color，但是在v5版本中只能使用 --color
调整命令：
```json
"scripts": {
    "start": "cross-env NODE_ENV=dev webpack-dev-server --hot --progress --color  --config ./webpack.dev.js",
}
```
### devServer 中 disableHostCheck报错
```
devServer: {
    ... 
    disableHostCheck: true, 
    ... 
},
```
修改为：
```
devServer: {
    ... 
    allowedHosts: "all", 
    ... 
},
```
当设置为 'all' 时会跳过host检查。并不推荐这样做，因为不检查host的应用程序容易受到DNS重绑定攻击。
### vue-loader问题
注意 vue-loader不同的版本是对应VUE不同版本的，这里一定要注意，如果你的VUE版本是2.x那么你要使用vue-loader@15.x，如果是vue@3.x那么要使用vue-loader@16.x
```js
// vue-loader@16.x
// 对应vue@3.x
const { VueLoaderPlugin } = require('vue-loader/dist/index');
module.exports = {
    plugins: [
        new VueLoaderPlugin(),
    ]
}
// vue-loader@15.x
// 对应vue@2.x
const { VueLoaderPlugin } = require('vue-loader');
module.exports = {
    plugins: [
        new VueLoaderPlugin(),
    ]
}
```
### webpack@5.x与vue-loader@16.x版本中间的一个报错问题,DescriptionDataMatcherRulePlugin | webpack5 报错问题
解决方案
1. DescriptionDataMatcherRulePlugin是出现在vue-loader里面的
2. webpack@5里面为啥不见了，我去webpack源码里找，竟然把文件名给改了，然后vue-loader那边没有同步修改
3. 解决方案：npm i webpack@5.44.0 -D 或者等vue-loader 更新
### postcss-loader 问题
```
npm install --save-dev autoprefixer
```
```js
// 新的配置
module.exports = {
    module: {
        rules: [
            {
                test: /\.(c|sa|sc)ss$/,
                use: [
                     'vue-style-loader', // <style></style> 插入页面,development下使用
                     'css-loader',
                     {
                        loader: 'postcss-loader',
                        options: {
                            postcssOptions: {
                                plugins: [
                                    [
                                        // "postcss-preset-env", // postcss-preset-env 包含autoprefixer （npm install postcss-preset-env --save-dev）
                                        // "postcss-nested",
                                        "autoprefixer", // css 加前缀
                                    ],
                                ],
                            },
                        }
                    },
                    'sass-loader'
                     
                ]
            }
        ]
    }
}
```
### optimization 配置
production
```js
module.exports = {
    optimization: {
        // 告知 webpack 当选择模块 id 时需要使用哪种算法
        moduleIds: 'deterministic' // 被哈希转化成的小位数值模块名。
    },
}
```
### webpack-manifest-plugin
```js
const { WebpackManifestPlugin } = require('webpack-manifest-plugin');
new WebpackManifestPlugin({
    publicPath: '',
    filter: function (FileDescriptor) {
        return FileDescriptor.isChunk;
    }
})
```