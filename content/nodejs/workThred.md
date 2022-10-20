---
title: "worker_threads 工作线程"
date: 2022-10-21T02:02:33+08:00
draft: false

categories: ["Node.js"]
---

# worker_threads 工作线程

Node.js 提供了 worker_threads, 在计算密集型的情况下很好使用

以下例子展示了工作线程的基本使用以及主进程和线程之间的数据交互

```js
const {
  Worker,
  isMainThread,
  parentPort,
  workerData,
} = require('worker_threads');

function fibo(n) {
  return n > 1 ? fibo(n - 1) + fibo(n - 2) : 1;
}

if (isMainThread) {
  const worker = new Worker(__filename, { workerData: 44 });
  const worker2 = new Worker(__filename, { workerData: 42 });
  worker.once('message', (message) => {
    console.log(`worker 1 res: ${message.res}`);
  });
  worker2.once('message', (message) => {
    console.log(`worker 2 res: ${message.res}`);
  });
} else {
  console.log(process.pid);
  parentPort.postMessage({ res: fibo(workerData) });
}
```
