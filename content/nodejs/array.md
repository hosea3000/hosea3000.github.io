---
title: "Array"
date: 2022-10-21T02:02:33+08:00
draft: false

categories: ["Node.js"]
---

### Js 判斷對象是否為數組

1. ECMAScript5 中有一個現成的方法：Array.isArray()

```js
var obj = { 1: [1], 2: [2] },
  arr = [1],
  str = '1';
Array.isArray(obj); // return false
Array.isArray(arr); // return true
Array.isArray(str); // return false
```

2. instanceof

   > instanceof 操作符用於判斷對象是不是類的實例。使用方法是 object instanceof Class

```js
var obj = { 1: [1], 2: [2] },
  arr = [1],
  str = '1';
obj instanceof Array; // return false
arr instanceof Array; // return true
str instanceof Array; // return false
```

3. 原型

   > 所有 Javascript 對象都派生自 Object 對象，可以通過 call 方法調用 Object 的 prototype 屬性判斷對象是不是數組

```js
var obj = { 1: [1], 2: [2] },
  arr = [1],
  str = '1';
Object.prototype.toString.call(obj); //return "[object Object]"
Object.prototype.toString.call(arr); //return "[object Array]"
Object.prototype.toString.call(str); //return "[object String]"
```

