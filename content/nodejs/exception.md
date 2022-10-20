---
title: "Node.js 中的异常捕获"
date: 2022-10-21T02:02:33+08:00
draft: false

categories: ["Node.js"]
---

## Node.js 中的异常捕获

### 同步代码的错误处理

1. 同步代码中的异常使用 try{}catch 结构即可捕获处理。

```js
try {
  throw new Error('错误信息');
} catch (e) {
  console.error(e.message);
}
```

### 异步代码的错误处理

1. callback 传参

```js
fs.mkdir('/dir', function (e) {
  if (e) {
    /*处理异常*/
    console.log(e.message);
  } else {
    console.log('创建目录成功');
  }
});
```

2. event 方式

```js
const events = require('events');

//创建一个事件监听对象
const emitter = new events.EventEmitter();

//监听error事件
emitter.addListener('error', function (e) {
  /*处理异常*/
  console.log(e.message);
});

//触发error事件
emitter.emit('error', new Error('出错啦'));
```

3. Promise 方式

```js
new Promise((resolve, reject) => {
  reject('error message');
}).catch((e) => {
  console.log(e.message);
});
```

4. async/await 方式

```js
function sleep() {
  return new Promise(function (resolve, reject) {
    reject('error message');
  });
}

async function main() {
  try {
    await sleep();
  } catch (err) {
    console.log(err);
  }
}

main().then();
```

5. process 方式

> 你的 node.js 进程都应该加上这两个时间的捕获做为你的最后补救措施，但是不应该当作 On Error Resume Next（出了错误就恢复让它继续）的等价机制。

npm `graceful` 专门做这个事情, 也可以引用这个包

```js
process.on('uncaughtException', function (e) {
  console.log(e.message);
});

process.on('unhandledRejection', function (e) {
  console.log(e.message);
});
```
