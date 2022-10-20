# js 中 sort 数字排序问题


# [js 中 sort 数字排序问题](https://www.cnblogs.com/fanda/p/4767984.html)

> **语法：arrayObject.sort(sortby)；参数 sortby 可选。规定排序顺序。必须是函数。**

- sort() 方法用于对数组的元素进行排序。默认按照字符进行排序, 就算是数字,也会先转成字符串进行排序
- sortby 参数是一个 function, 比较函数应该具有两个参数 a 和 b，其返回值如下：
- 若 a 小于 b，在排序后的数组中 a 应该出现在 b 之前，则返回一个小于 0 的值。
- 若 a 等于 b，则返回 0。
- 若 a 大于 b，则返回一个大于 0 的值。

### 用 js 中的 sort()方法排序数字(**注意这里是按照字符排序的**)

```js
var arr = [23, 12, 1, 34, 116, 8, 18, 37, 56, 50];
console.log(arr.sort());
// [1, 116, 12, 18, 23, 34, 37, 50, 56, 8]
```

### 自定义排序方法

```js
const arr = [23, 12, 1, 34, 116, 8, 18, 37, 56, 50];
console.log(arr.sort((a, b) => a - b));
// [1, 8, 12, 18, 23, 34, 37, 50, 56, 116]
```

### 默认的字符排序

```js
var arr = ['fanda', 'banner', 'find', 'zoom', 'index', 'width', 'javascript'];
console.log(arr.sort());
// ["banner", "fanda", "find", "index", "javascript", "width", "zoom"]
```

