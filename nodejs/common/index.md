# Node.js 资料收集区


# Node.js 资料收集区

[五月君的面试题](https://interview.nodejs.red/)

[如何分析 Node.js 中的内存泄漏](https://zhuanlan.zhihu.com/p/25736931)

## Node.js 的特点

- 基于 V8 的 Javascript 服务端 Runtime

- 事件驱动 (event loop)

- 异步非阻塞 I/O (通过线程池来实现)

I/O 和 CPU 完全分离, 实现非阻塞 I/O, 保证 Event loop 高效

## async/await 异步中异常捕获的问题

### 问题: 在如下场景中 try/catch 是否能够成功捕获错误

```js
async function test() {
  // throw new Error('Test Error');
  return Promise.reject('Test Error');
}

async function handle() {
  try {
    // case 1:  能捕获
    const res = await test();
    return res;

    // case 2: 能捕获
    return await test();

    // case 3: 不能捕获
    return test();
  } catch (err) {
    console.log('成功捕获 Error', err.toString());
  }
}

handle()
  .then()
  .catch((err) => {
    console.log('未捕获到 Error', err.toString());
  });
```

### 结论: 在 try 中异步方法调用前有 await 就能捕获到错误

### IO 多路复用

设计高并发系统?

#### 多线程

- 原理: 多线程, 每个线程处理一个请求(RPC 框架很多这样实现的),
- 问题: - 多线程存在上下文切换开销大的问题 - 多线程需要的内存资源比较多, 会有瓶颈

#### 单线程

用单线程处理网络连接

> 怎么保证数据不丢失, DMA

> 用户态? 内核态?
