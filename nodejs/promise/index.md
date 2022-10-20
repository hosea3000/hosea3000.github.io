# 手动实现 Promise


# 手动实现 Promise

> 参考链接: [史上最最最详细的手写 Promise 教程](https://juejin.im/post/5b2f02cd5188252b937548ab)

在论坛上看到很多手写 promise 的帖子, 是一个常见的面试题, 自己也尝试实现一下, 同时可以帮助自己更好的理解 Promise.

## 基本实现

- Promise 是一个类
- new Promise 时，会返回一个 promise 的对象，它会传一个执行器（executor），这个执行器是立即执行的
- 另外每个 promise 实例上都会有一个 then 方法，参数分别是成功（有成功的值）和失败（有失败的原用）两个方法
- promise 有三个状态：成功态(fulfilled)，失败态(rejected)，等待态(pending)。
- 默认状态是等待态，等待态可以变成成功态或失败态
- 一旦成功或失败就不能再变会其他状态了 知道这些就可以先简单实现一下了

```js
class PromiseA {
  constructor(exec) {
    // ['pending', 'fulfilled', 'rejected']
    this.state = 'pending';
    this.value = undefined;
    this.reason = undefined;
    const resolve = (value) => {
      if (this.state === 'pending') {
        this.value = value;
        this.state = 'fulfilled';
      }
    };
    const reject = (reason) => {
      if (this.state === 'pending') {
        this.reason = reason;
        this.state = 'rejected';
      }
    };
    try {
      exec(resolve, reject);
    } catch (err) {
      reject(err);
    }
  }

  then(onFulfilled, onRejected) {
    if (this.state === 'fulfilled') {
      onFulfilled(this.value);
    }
    if (this.state === 'rejected') {
      onRejected(this.reason);
    }
  }
}

const a = new PromiseA((resolve) => {
  resolve(1);
});

a.then(
  (res) => {
    console.log(res);
    console.log('end');
  },
  (err) => {
    console.log(err);
  }
);
```

### 异步调用

> 上面简单的实现了 Promise, 但是当 resolve 在 setTomeout 内执行，调用 then 时 state 还是 pending 等待状态, 这样上面的实现就不好用了. 解决办法是新建两个数组, 用来保存成功或者失败的执行函数, 当 Promise 不再为 pending 状态时调用他们. (为什是数组? 因为 Promise 可以多次调用 then )

```js
// 多个then的情况
const p = new Promise();
p.then();
p.then();
```

```js
// 增加异步调用的实现
class PromiseA {
  constructor(exec) {
    // ['pending', 'fulfilled', 'rejected']
    this.state = 'pending';
    this.value = undefined;
    this.reason = undefined;
    // 成功存放的数组
    this.onResolvedCallbacks = [];
    // 失败存放法数组
    this.onRejectedCallbacks = [];
    const resolve = (value) => {
      if (this.state === 'pending') {
        this.value = value;
        this.state = 'fulfilled';
        this.onResolvedCallbacks.forEach((fn) => fn());
      }
    };
    const reject = (reason) => {
      if (this.state === 'pending') {
        this.reason = reason;
        this.state = 'rejected';
        this.onRejectedCallbacks.forEach((fn) => fn());
      }
    };
    try {
      exec(resolve, reject);
    } catch (err) {
      reject(err);
    }
  }

  then(onFulfilled, onRejected) {
    // 当状态state为pending时
    if (this.state === 'pending') {
      // onFulfilled传入到成功数组
      this.onResolvedCallbacks.push(() => {
        onFulfilled(this.value);
      });
      // onRejected传入到失败数组
      this.onRejectedCallbacks.push(() => {
        onRejected(this.reason);
      });
    }
    if (this.state === 'fulfilled') {
      onFulfilled(this.value);
    }
    if (this.state === 'rejected') {
      onRejected(this.reason);
    }
  }
}
```

### 链式调用

**`new Promise().then().then()`** 这就是链式调用, 实现链式调用的主要作用就是可以解决回调地狱

1 为了达成链式，我们默认在第一个 then 里返回一个 promise。就是在 then 里面返回一个新的 promise, 称为 promise2：

- 将这个 promise2 返回的值传递到下一个 then 中
- 如果返回一个普通的值，则将普通的值传递给下一个 then 中

```js
class PromiseA {
  constructor(exec) {
    // ['pending', 'fulfilled', 'rejected']
    this.state = 'pending';
    this.value = undefined;
    this.reason = undefined;
    // 成功存放的数组
    this.onResolvedCallbacks = [];
    // 失败存放法数组
    this.onRejectedCallbacks = [];
    const resolve = (value) => {
      if (this.state === 'pending') {
        this.value = value;
        this.state = 'fulfilled';
        this.onResolvedCallbacks.forEach((fn) => fn());
      }
    };
    const reject = (reason) => {
      if (this.state === 'pending') {
        this.reason = reason;
        this.state = 'rejected';
        this.onRejectedCallbacks.forEach((fn) => fn());
      }
    };
    try {
      exec(resolve, reject);
    } catch (err) {
      reject(err);
    }
  }

  then(onFulfilled, onRejected) {
    const promise2 = new PromiseA((resolve, reject) => {
      // 当状态state为pending时
      if (this.state === 'pending') {
        // onFulfilled传入到成功数组
        this.onResolvedCallbacks.push(() => {
          const x = onFulfilled(this.value);
          // resolvePromise函数，处理自己return的promise和默认的promise2的关系
          resolvePromise(promise2, x, resolve, reject);
        });
        // onRejected传入到失败数组
        this.onRejectedCallbacks.push(() => {
          const x = onRejected(this.reason);
          resolvePromise(promise2, x, resolve, reject);
        });
      }
      if (this.state === 'fulfilled') {
        const x = onFulfilled(this.value);
        resolvePromise(promise2, x, resolve, reject);
      }
      if (this.state === 'rejected') {
        const x = onRejected(this.reason);
        resolvePromise(promise2, x, resolve, reject);
      }
    });
    return promise2;
  }
}
```

#### 实现 resolvePromise 函数

- Otherwise, if x is an object or function,Let then be x.then
- x 不能是 null
- x 是普通值 直接 resolve(x)
- x 是对象或者函数（包括 promise），let then = x.then

```js
function resolvePromise(promise2, x, resolve, reject) {
  // 循环引用报错
  if (x === promise2) {
    // reject报错
    return reject(new TypeError('Chaining cycle detected for promise'));
  }
  // 防止多次调用
  let called;
  // x不是null 且x是对象或者函数
  if (x != null && (typeof x === 'object' || typeof x === 'function')) {
    try {
      // A+规定，声明then = x的then方法
      const { then } = x;
      // 如果then是函数，就默认是promise了
      if (typeof then === 'function') {
        // 就让then执行 第一个参数是this   后面是成功的回调 和 失败的回调
        then.call(
          (x, y) => {
            // 成功和失败只能调用一个
            if (called) return;
            called = true;
            // resolve的结果依旧是promise 那就继续解析
            resolvePromise(promise2, y, resolve, reject);
          },
          (err) => {
            // 成功和失败只能调用一个
            if (called) return;
            called = true;
            reject(err); // 失败了就失败了
          }
        );
      } else {
        resolve(x); // 直接成功即可
      }
    } catch (e) {
      // 也属于失败
      if (called) return;
      called = true;
      // 取then出错了那就不要在继续执行了
      reject(e);
    }
  } else {
    resolve(x);
  }
}
```

