---
title: "Express 错误处理中间件如何捕获 async 方法的异常?"
date: 2022-10-21T02:02:33+08:00
draft: false

categories: ["Node.js"]
---

### 问题示例

```js
const express = require('express');
const app = express();

app.get('/', async (req, res, next) => {
  throw new Error('test error');
});

app.use(async function (error, req, res, next) {
  console.log('will not print'); // 这里无法捕获到错误, 关键点在于action是异步方法
  res.json({ message: error.message || error });
});

const server = app.listen(3000, function () {
  console.log('server started...');
});
```

定义的错误中间件无法捕获到 action 抛出的错误, 解决办法 如下:

### 解决方法一

不要直接抛错, 用 next()

```js
app.get('/', async (req, res, next) => {
  next(new Error('test error'));
});
```

### 解决办法二

自定义一个 action 的处理函数

```js
const express = require('express');
const app = express();

const asyncMiddleware = (fn) => async (req, res, next) => {
  try {
    const data = await fn(req, res, next);
    return data;
  } catch (err) {
    next(err);
  }
};

app.get(
  '/',
  asyncMiddleware(async (req, res, next) => {
    throw new Error('test error');
  })
);

app.use(async function (error, req, res, next) {
  console.log('will print');
  res.json({ message: error.message || error });
});

const server = app.listen(3000, function () {
  console.log('server started...');
});
```

### 解决办法三

利用第三方包 _express-async-errors_

```js
const express = require('express');
const app = express();
require('express-async-errors');

app.get('/', async (req, res, next) => {
  throw new Error('test error');
});

app.use(async function (error, req, res, next) {
  console.log('will print');
  res.json({ message: error.message || error });
});

const server = app.listen(3000, function () {
  const host = server.address().address;
  const port = server.address().port;

  console.log('server start', host, port);
});
```
