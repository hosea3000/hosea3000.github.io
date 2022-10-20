---
title: "Node.js 的模块机制"
date: 2022-10-21T02:02:33+08:00
draft: false

categories: ["Node.js"]
---

# Node.js 的模块机制

Node.js 在实现模块系统时采用了 CommonJS 的模块规范。

CommonJS 的模块规范分为 3 个部分：

1. 模块引用：通过 require()方法并传入一个模块标识来引入一个模块的 API 到当前上下文中，如 var math = require('math');

2. 模块定义：通过 exports 对象来导出当前模块的方法或变量。模块中还存在一个 module 对象，exports 实际上是 module 的属性。在 Node 中，一个文件就是一个模块，模块内的“全局变量”对外都不可见，只有挂载在 exports 上的属性才是公开的，如 exports.add = function() {}; exports.PI = 3.1415926;

3. 模块标识：模块标识其实就是传递给 require()方法的参数，它必须是小驼峰命名的字符串，或者是.、..开头的相对路径，或者绝对路径。它可以没有文件名后缀.

每个模块具有独立的空间，它们互不干扰，导出和引用都很简单。CommonJs 构建的这套模块导出和引入机制使得用户完全不必考虑变量污染的问题。
