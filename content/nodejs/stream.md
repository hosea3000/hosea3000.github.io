---
title: "NodeJS 自定义流的实现"
date: 2022-10-21T02:02:33+08:00
draft: false

categories: ["Node.js"]
---

## NodeJS 自定义流的实现

### 概述

常见的自定义流有四种，Readable（可读流）、Writable（可写流）、Duplex（双工流）和 Transform（转换流），常见的自定义流应用有 HTTP 请求、响应，crypto 加密，进程 stdin 通信等等。

### 自定义可读流

```js
// 创建自定义可读流
const { Readable } = require('stream');

// 创建自定义可读流的类
class MyRead extends Readable {
  constructor() {
    super();
    this.index = 0;
  }

  // 重写自定义的可读流的 _read 方法
  // 注意这个方法会调用多次
  _read() {
    this.index++;
    this.push(this.index + '');

    if (this.index === 3) {
      this.push(null);
    }
  }
}

// 验证自定义可读流
let myRead = new MyRead();

myRead.on('data', (data) => {
  console.log(data);
});

myRead.on('end', function () {
  console.log('读取完成');
});

// <Buffer 31>
// <Buffer 32>
// <Buffer 33>
// 读取完成
```

我们自己写的 `_read` 方法会先查找并执行，在读取时使用 push 方法将数据读取出来，直到 push 的值为 null 才会停止，否则会认为没有读取完成，会继续调用 `_read`。

### 自定义可写流

```js
// 创建自定义可写流
const { Writable } = require("stream");

// 创建自定义可写流的类
class MyWrite extends Writable {
    // 重写自定义的可写流的 _write 方法
    _write(chunk, encoding, callback)) {
        callback(); // 将缓存区写入文件
    }
}

// 验证自定义可写流
let myWrite = new MyWrite();

myWrite.write("hello", "utf8", () => {
    console.log("hello ok");
});

myWrite.write("world", "utf8", () => {
    console.log("world ok");
});

// hello ok
// world ok
```

### 自定义双工流 Duplex

双工流分别具备 Readable 和 Writable 的功能，但是读和写互不影响，互不关联。

```js
// 创建自定双工流
const { Duplex } = require("stream");

// 创建自定义双工流的类
class MyDuplex extends Duplex {

    // 重写自定义的双工流的 _read 方法
    _read() {
        this.push("123");
        this.push(null);
    }

    // 重写自定义的双工流的 _write 方法
    _write(chunk, encoding, callback)) {
        callback();
    }
}


// 验证自定义双工流
let myDuplex = new MyDuplex();

myDuplex.on("readable", () => {
    console.log(myDuplex.read(1), "----");
});

setTimeout(() => {
    myDuplex.on("data", data => {
        console.log(data, "xxxx");
    });
}, 3000);

// <Buffer 31> ----
// <Buffer 32> xxxx
// <Buffer 32> ----
// <Buffer 33> xxxx
```

如果 readable 和 data 两种读取方式都使用默认先通过 data 事件读取，所以一般只选择一个，不要同时使用，可读流的特点是读取数据被消耗掉后就丢失了（缓存区被清空），如果非要两个都用可以加一个定时器（绝对不要这样写）。

### 自定义转化流 Transform

转化流的意思是即可以当作可读流，又可以当作可写流，创建一个类名为 MyTransform，并继承 stream 中的 Transform 类，重写 \_transform 方法，该方法的参数和 \_write 相同。

在自定义转化流的 \_transform 方法中，读取数据的 push 方法和 写入数据的 callback 都可以使用。

由此可以看出，Transform 类型可以将可读流转化为可写流，也可以将可写流转化成可读流，他的主要目的不是像其他类型的流一样负责数据的读写，而是既作为可读流又作为可写流，实现流的转化，即实现对数据的特殊处理，如 zib 模块实现的压缩流，cropo 模块实现的加密流，本质都是转化流，将转化流作为可写流，将存储文件内容的可写流通过 pipe 方法写入转化流，再将转化流作为可读流通过 pipe 方法将处理后的数据响应给浏览器。

```js
// 创建自定义转化流
const { Transform } = require('stream');

// 创建自定义转化流的类
class MyTransform extends Transform {

    // 重写自定义的转化流的 _transform 方法
    _transform(chunk, encoding, callback)) {
        console.log(chunck.toString.toUpperCase());
        callback();
        this.push('123');
    }
}

// 验证自定义转化流
let myTransForm = new MyTransform();

// 使用标准输入
process.stdin.pipe(myTransForm).pipe(process.stdin);
```
