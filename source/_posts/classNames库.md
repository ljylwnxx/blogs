---
title: classNames库
date: 2023-12-18 21:45:38
tags: classNames库
categories: 工具
---

# 简述

classNames 是一个 JavaScript 库,用于动态生成 CSS 类名，根据不同的条件添加或移除 CSS 类名。简单来说就是动态地去操作类名，把符合条件的类名粘在一起，简化在 React 组件中处理类名的复杂逻辑。

# 安装（使用 npm）

```
npm install classnames
```

# 引入

## 在 nodejs 里引入

```
var classNames = require('classnames')
```

## 在 js 里引入

```
import classnames from 'classnames'
```

# 基本使用

classNames 函数接受任意数量的参数，可以是 string、boolean、number、array 等。

## 普通字符串粘合

将参数拼接为字符串，中间用空格分开

```
classNames('foo', 'bar') // => 'foo bar'
```

## 带条件的类参数

这里第二个参数是对象类型，键值为 true，则粘合进 className 里

```
classNames('foo', { bar: true }) // => 'foo bar'
```

若为 false，则不粘进去

```
classNames('foo', { bar: false }) // => 'foo'
```

## 参数类型是数组

```
var arr = ['b', { c: true, d: false }]
classNames('a', arr) // => 'a b c'
```

## 特别注意

null 和 undefiend 会被忽略

```
classNames(null, false, 'bar', undefined, 0, 1, { baz: null }, '') // => 'bar 1'
```

# 在 React 中优雅地使用 classnames

```
import classnames from 'classnames'

const Button = ({primary,size})=>{
  const classes = classnames('btn', {
    'btn-primary':primary,
    'btn-large': size==='large',
    'btn-small': size==='small'
  })
  return <button className= {classes}>点击按钮</button>
}
```

在上面的示例中，classnames 函数接受一个类名和一个将类名映射为布尔值的对象。
classes 就可以变成一个对象，通过键值的条件确定最终生成的 className。

- 如果给定类名的布尔值为 true，则该类名将包含在类名的最终列表中。
- 如果给定类名的布尔值为 false，则不包括类名。
  另外可以将字符串作为第二个参数传递给类名，如果给定类名的布尔值为 true，那么它将被添加到类名的最终列表中。

```
const classes = classnames('btn', primary && 'btn-primary')
```

# 原理

## classNames 源码

```
(function () {
	'use strict'
  //hasOwnProperty这个方法用来判断对象的属性是否属于自己本身——而不是往原型链上面找到的
	var hasOwn = {}.hasOwnProperty

	function classNames () {
    //用于存储生成的类名
		var classes = ''
    //遍历classnames的所有参数
		for (var i = 0; i < arguments.length; i++) {
      //遍历 arguments 拿到每一项
			var arg = arguments[i]
			if (arg) {
				classes = appendClass(classes, parseValue(arg))
			}
		}

		return classes
	}

	function parseValue (arg) {
    //获取每一个参数的类型
    //如果是字符串或数字就直接加到classes数组里
		if (typeof arg === 'string' || typeof arg === 'number') {
			return arg
		}

		if (typeof arg !== 'object') {
			return ''
		}
    //如果参数是数组，则将数组的值当作参数，调用自己
		if (Array.isArray(arg)) {
			return classNames.apply(null, arg)
		}
    //如果是对象且有自定义的toString方法，则调用toString方法添加到classes对象里
		if (arg.toString !== Object.prototype.toString && !arg.toString.toString().includes('[native code]')) {
			return arg.toString()
		}

		var classes = ''
    //对于普通对象类型，则遍历对象的每一组键值对
		for (var key in arg) {
      //将值为true的键作为className插入classes数组里
			if (hasOwn.call(arg, key) && arg[key]) {
				classes = appendClass(classes, key)
			}
		}

		return classes
	}

	function appendClass (value, newClass) {
		if (!newClass) {
			return value
		}

		if (value) {
			return value + ' ' + newClass
		}

		return value + newClass
	}

	if (typeof module !== 'undefined' && module.exports) {
		classNames.default = classNames
		module.exports = classNames
	} else if (typeof define === 'function' && typeof define.amd === 'object' && define.amd) {
		// register as 'classnames', consistent with npm package name
		define('classnames', [], function () {
			return classNames
		});
	} else {
		window.classNames = classNames
	}
}())
```

## 源码分析

首先分析 index.js，采用了自执行的函数，避免变量污染冲突，采用严格模式。

1. 函数声明了一个名为 classes 的空数组，该数组将用于存储生成的类名。
2. 然后，for 循环函数自带 arguments 实例，得到 classnames 函数的所有实参。
   对于每个参数，执行以下步骤:

- 检查参数的类型。
  - 如果参数是一个字符串或数字，添加到类数组。
  - 如果参数是数组，则函数检查它是否为非空。如果是，则函数以数组元素作为参数递归地调用自身，并将结果添加到类数组中。
  - 如果参数是一个对象，那么先判定它的 toString 方法是否重写了。
    - 如果是重写了，直接将调用 toString 的结果添加到类数组中。
    - 如果不是，函数将遍历对象自己的可枚举属性，并将属性名称添加到类数组中(如果它们的对应值为真)。

3. 提供了三种导出 classNames 的方式。

- module.exports 导出 classNames
- 直接返回 classNames
- 将 classNames 挂载到 window 上

4. 循环结束后，将类数组合并到一个单独的字符串中，并返回结果字符串。

## 疑问

1. arg.toString !== Object.prototype.toString 是什么？

首先所有 js 对象都继承 Object 对象，都继承 toString 方法，这个表达式就是这个对象的 toString 方法不是继承自 Object.prototype

2. arg.toString.toString().includes("[native code]") 是什么意思？

首先要知道 toString 方法的一些知识：

- 如果对一个自定义函数调用 toString()方法时，可以得到该函数的源代码。
- 如果对内置函数调用 toString()方法时，会得到一个'[native code]'字符串。
  因此，可以使用 toString()方法来区分自定义函数和内置函数。
  注意：上诉表达式中是对函数调用 toString()方法。
  通过对 toString 函数调用 toString 方法，就可以判断这个 toString 是内置的，还是自定义的。

```
arg.toString !== Object.prototype.toString && !arg.toString.toString().includes('[native code])
```

这里两个表达式连起来的意思就是：现在这个类有 toString 方法，而且还是自定义的

## 使用示例

从源码来看，classNames 能够接入的参数是多种多样的，下面列举一些。

```
//字符串(能够单个，多个字符串)
classNames('el-checkbox-button--m', "is-disabled")

//对象(能够单个，多个)
classNames({"is-disabled"：true}, {'is-disabled': disabled})

//字符串+对象
classNames('el-checkbox-button--m', {"is-disabled"：true, 'is-disabled': disabled})

//数组（数组项能够是字符串，对象）
classNames(['el-checkbox-button--m', "is-disabled"])
classNames(['el-checkbox-button--m', {"is-disabled"：true，'is-disabled': disabled}])
```
