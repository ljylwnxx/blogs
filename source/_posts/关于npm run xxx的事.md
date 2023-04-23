---
title: 关于npm run xxx的事
date: 2023-04-23 10:27:08
tags: npm 
categories: npm
---
大家在每天开始一天新的工作的时候，第一件事就是打开vs code，然后就在命令行输入npm run start,emmm...之后再等待个一小会，项目就跑起来了，就开始愉快的一天了。
那问题来了，你想过npm run xxx之后发生了什么吗？
几乎每一个前端项目都一个package.json文件，这个文件里面总是能看到一个关键字 scripts
```
{
  ...,
 "scripts": {
   "webpack": "webpack --config webpack.config.js",
   "postinstall": "node ./scripts/postinstall.js"
 },
  ...,
}
```
我们可以通过 npm run xxx 来执行 scripts 中的命令。例如，当我们执行 npm run webpack 时，实际上是在执行 scripts 中的 webpack 命令，即 webpack --config webpack.config.js
# 一、npm  run xxx 的时候发生了什么？
npm run xxx 的时候，首先会去项目的 package.json 文件里找 scripts 里找对应的 xxx，然后执行 xxx 的命令，例如启动 vue 项目 npm run serve 的时候，实际上就是执行了 vue-cli-service serve 这条命令。
package.json 文件
```
  "scripts": {
    "serve": "vue-cli-service serve",
    "build": "vue-cli-service build",
    "lint": "vue-cli-service lint"
  },
```
# 二、那为什么不直接执行vue-cli-service serve而要执行npm run serve 呢？
因为直接执行vue-cli-service serve，会报错，因为操作系统中没有存在vue-cli-service这一条指令
既然vue-cli-service这条指令不存在操作系统中，为什么执行npm run serve的时候，也就是相当于执行了vue-cli-service serve ，为什么这样它就能成功，而且不报指令不存在的错误呢？
# 三、为什么执行 npm run serve 的能成功？
在我们在安装依赖的时候，是通过npm i xxx 来执行的，例如 npm i @vue/cli-service，npm 在 安装这个依赖的时候，就会node_modules/.bin/ 目录中创建 好vue-cli-service 为名的几个可执行文件了。
.bin 目录，这个目录不是任何一个 npm 包。目录下的文件，表示这是一个个软链接，打开文件可以看到文件顶部写着 #!/bin/sh ，表示这是一个脚本。
当使用 npm run serve 执行 vue-cli-service serve 时，虽然没有安装 vue-cli-service的全局命令，但是 npm 会到 ./node_modules/.bin 中找到vue-cli-service 文件作为脚本来执行。
# 四、你说.bin 目录下的文件表示软连接，那这个bin目录下的那些软连接文件是哪里来的呢？它又是怎么知道这条软连接是执行哪里的呢？
从 package-lock.json 中可知，当我们 npm i 整个新建的 vue 项目的时候，npm 将 bin/vue-cli-service.js 作为 bin 声明了
所以在 npm install 时，npm 读到该配置后，就将该文件软链接到 ./node_modules/.bin 目录下，而 npm 还会自动把 node_modules/.bin 加入$PATH，这样就可以直接作为命令运行依赖程序和开发依赖程序，不用全局安装了
假如我们在安装包时，使用  npm install -g xxx  来安装，那么会将其中的 bin 文件加入到全局
npm i 的时候，npm 就帮我们把这种软连接配置好了，其实这种软连接相当于一种映射，执行 npm run xxx 的时候，就会到 node_modules/bin 中找对应的映射文件，然后再找到相应的 js 文件来执行。
# 五、为什么在 node_modules/bin 中 有三个 vue-cli-service 文件？
如果我们在 cmd 里运行的时候，windows 一般是调用了 vue-cli-service.cmd，这个文件，这是 windows 下的批处理脚本：
```
@ECHO off
GOTO start
:find_dp0
SET dp0=%~dp0
EXIT /b
:start
SETLOCAL
CALL :find_dp0
IF EXIST "%dp0%\node.exe" (
  SET "_prog=%dp0%\node.exe"
) ELSE (
  SET "_prog=node"
  SET PATHEXT=%PATHEXT:;.JS;=;%
)
endLocal & goto #_undefined_# 2>NUL || title %COMSPEC% & "%_prog%"  "%dp0%\..\@vue\cli-service\bin\vue-cli-service.js" %*
```
所以当我们运行vue-cli-service serve这条命令的时候，就相当于运行 node_modules/.bin/vue-cli-service.cmd serve。
然后这个脚本会使用 node 去运行 vue-cli-service.js这个 js 文件
由于 node 中可以使用一系列系统相关的 api ，所以在这个 js 中可以做很多事情，例如读取并分析运行这条命令的目录下的文件，根据模板生成文件等。
```
# unix 系默认的可执行文件，必须输入完整文件名
vue-cli-service
# windows cmd 中默认的可执行文件，当我们不添加后缀名时，自动根据 pathext 查找文件
vue-cli-service.cmd
# Windows PowerShell 中可执行文件，可以跨平台
vue-cli-service.ps1
```
# 总结
执行npm run xxx时，会先从当前目录下的node_modules/.bin中去查找对应的可执行程序执行；
如果无法找到，就会在npm的全局安装路径进行查找，也就是npm i -g xxx时安装的路径；
如果还找不到，就会从系统环境变量中查找；
再找不到就会报错了；