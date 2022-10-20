# 进程和线程


### 进程和线程

进程: 系统进行资源分配和调度的基本单位
线程: 程序执行的最小单位(CPU 调度的最小单位)
进程是线程的容器, 一个进程可以拥有多个线程,
每个进程都拥有自己的独立空间地址、数据栈
一个进程无法访问另外一个进程里定义的变量、数据结构，只有建立了 IPC 通信，进程之间才可数据共享

### Javascript 的单线程

Javascript 就是属于单线程，程序顺序执行，可以想象一下队列，前面一个执行完之后，后面才可以执行，当你在使用单线程语言编码时切勿有过多耗时的同步操作，否则线程会造成阻塞，导致后续响应无法处理。你如果采用 Javascript 进行编码时候，请尽可能的使用异步操作。

#### CPU 计算阻塞的例子

```js
const computation = () => {
  let sum = 0;
  console.info('计算开始');
  console.time('计算耗时');

  for (let i = 0; i < 1e10; i += 1) {
    sum += i;
  }

  console.info('计算结束');
  console.timeEnd('计算耗时');
  return sum;
};
```

因为上面的例子是 CPU 计算, 所以耗时时间比较长, 下面拿这个方法来测试 CPU 计算阻塞 WEB 服务的例子

```js
// compute.js
const computation = () => {
  let sum = 0;
  console.info('计算开始');
  console.time('计算耗时');

  for (let i = 0; i < 1e10; i += 1) {
    sum += i;
  }

  console.info('计算结束');
  console.timeEnd('计算耗时');
  return sum;
};

module.exports = { computation };
```

再启用一个 web 服务

```js
// index.js
const http = require('http');
const { computation } = require('./compute');

const server = http.createServer((req, res) => {
  if (req.url === '/compute') {
    computation();
    res.end('compute ok');
  }
  if (req.url === '/ping') {
    res.end('req pong');
  }
});

server.listen(3000, () => {
  console.log('http start, listen on port 3000');
});
```

启动服务开始测试:

1. 访问链接: `http://localhost:3000/ping` 会很快得到服务器响应 `req pong`
2. 访问链接: `http://localhost:3000/compute` 服务器开始计算, 短时间内无响应
3. 再次访问链接: `http://localhost:3000/ping` 也会被卡主
4. 等待链接: `http://localhost:3000/compute` 计算完成, 正确返回
5. 再次访问链接: `http://localhost:3000/ping` 能正常访问

之前我一直有一个误区, 认为`将这些计算耗时的操作异步去执行就不会阻塞WEB服务`. 显然这样是错误的认知, 这是将异步和非阻塞两个概念弄混了. 异步调用这个方法, 只会改变计算的时间片(也就是在不同的事件循环中执行而已), 而计算还是在主线程上执行.

结论:
Node.js 虽然是单线程模型，但是其基于事件驱动、异步非阻塞模式，可以应用于高并发场景，避免了线程创建、线程之间上下文切换所产生的资源开销。但是不合适`CPU密集`的场景

### 多进程

在 CPU 密集型的业务场景下, 单进程就不适用了, 可以考虑多进程
针对上一个问题可以考虑再起一个进程去执行计算, 等到计算完成通知主进程

```js
// compute.js
const computation = () => {
  let sum = 0;
  console.info('计算开始');
  console.time('计算耗时');

  for (let i = 0; i < 1e10; i += 1) {
    sum += i;
  }

  console.info('计算结束');
  console.timeEnd('计算耗时');
  return sum;
};

process.on('message', (msg) => {
  console.log(msg, 'process.pid', process.pid); // 子进程id
  const sum = computation();
  // 如果Node.js进程是通过进程间通信产生的，那么，process.send()方法可以用来给父进程发送消息
  process.send(sum);
});
```

主进程开启一个 http 服务, 当需要计算的请求进来是 fork 一个子进程计算, 计算完成返回给客户端, 其他请求不影响, 不会阻塞服务器

```js
const http = require('http');
const { fork } = require('child_process');

const server = http.createServer((req, res) => {
  if (req.url === '/compute') {
    const childProcess = fork('./compute');
    childProcess.send('开启一个新的子进程');

    // 当一个子进程使用 process.send() 发送消息时会触发 'message' 事件
    childProcess.on('message', (sum) => {
      res.end(`Sum is ${sum}`);
      childProcess.kill();
    });

    // 子进程监听到一些错误消息退出
    childProcess.on('close', (code, signal) => {
      console.log(
        `收到close事件，子进程收到信号 ${signal} 而终止，退出码 ${code}`
      );
      childProcess.kill();
      res.end('compute ok');
    });
  }
  if (req.url === '/ping') {
    res.end('req pong');
  }
});

server.listen(3000, () => {
  console.log('http start, listen on port 3000');
});
```

这样子实现的差别就是:
`compute` 接口等待服务器计算完成的响应时, `ping` 接口能够正确响应, 不收影响

多线程的代价还在于创建新的线程和执行期上下文线程的切换开销，由于每创建一个线程就会占用一定的内存，当应用程序并发大了之后，内存将会很快耗尽。类似于上面单线程模型中例举的例子，需要一定的计算会造成当前线程阻塞的，还是推荐使用多线程来处理

### Node.js 的多进程

Node.js 是 Javascript 在服务端的运行环境，构建在 chrome 的 V8 引擎之上，基于事件驱动、非阻塞 I/O 模型，充分利用操作系统提供的异步 I/O 进行多任务的执行，适合于 I/O 密集型的应用场景.

针对于多核 CPU 的服务器, 如果只启用一个进程,无疑是浪费了机器的性能. 多核 CPU 系统之上，可以用过 child_process.fork 开启多个进程（Node.js 在 v0.8 版本之后新增了 Cluster 来实现多进程架构） ，即 多进程 + 单线程 模式。
注意：`开启多进程不是为了解决高并发，主要是解决了单进程模式下 Node.js CPU 利用率不足的情况，充分利用多核 CPU 的性能`。

#### 创建进程的方式

- child_process.spawn()：适用于返回大量数据，例如图像处理，二进制数据处理。
- child_process.exec()：适用于小量数据，maxBuffer 默认值为 200 \* 1024 超出这个默认值将会导致程序崩溃，数据量过大可采用 spawn。
- child_process.execFile()：类似 child_process.exec()，区别是不能通过 shell 来执行，不支持像 I/O 重定向和文件查找这样的行为
- child_process.fork()： 衍生新的进程，进程之间是相互独立的，每个进程都有自己的 V8 实例、内存，系统资源是有限的，不建议衍生太多的子进程出来，通长根据系统 CPU 核心数设置。

```js
const { spawn, exec, execFile, fork } = require('child_process');

spawn('ls', ['-l'], { cwd: '/usr' });

exec('node -v', (error, stdout, stderr) => {
  console.log({ error, stdout, stderr }); // { error: null, stdout: 'v8.5.0\n', stderr: '' }
});

execFile('node', ['-v'], (error, stdout, stderr) => {
  console.log({ error, stdout, stderr }); // { error: null, stdout: 'v8.5.0\n', stderr: '' }
});

const childFork = fork('./http-server.js');
console.log(childFork.pid); // process.pid:  57252
```

### 创建多进程的 web 服务器

#### 主进程

master.js 主要处理以下逻辑：

- 创建一个 server 并监听 3000 端口。
- 根据系统 cpus 开启多个子进程
- 通过子进程对象的 send 方法发送消息到子进程进行通信
- 在主进程中监听了子进程的变化，如果是自杀信号重新启动一个工作进程。
- 主进程在监听到退出消息的时候，先退出子进程在退出主进程

```js
const cpus = require('os').cpus();
const { fork } = require('child_process');
const server = require('net').createServer();

server.listen(3000);
process.title = 'node-master';

const workers = {};
function createWorker() {
  const worker = fork('./worker.js');
  worker.on('message', (message) => {
    if (message.act === 'suicide') {
      createWorker();
    }
  });

  worker.on('exit', (code, signal) => {
    console.log('worker process exited, code: %s signal: %s', code, signal);
    delete workers[worker.pid];
  });

  worker.send('server', server);
  workers[worker.pid] = worker;
  console.log(
    'worker process created, pid: %s ppid: %s',
    worker.pid,
    process.pid
  );
}

for (let i = 0; i < cpus.length; i += 1) {
  createWorker();
}

function close(code) {
  console.log('进程退出！', code);

  if (code !== 0) {
    for (const pid of Object.keys(workers)) {
      console.log('master process exited, kill worker pid: ', pid);
      workers[pid].kill('SIGINT');
    }
  }

  process.exit(0);
}

process.once('SIGINT', close.bind(this, 'SIGINT')); // kill(2) Ctrl-C
process.once('SIGQUIT', close.bind(this, 'SIGQUIT')); // kill(3) Ctrl-\
process.once('SIGTERM', close.bind(this, 'SIGTERM')); // kill(15) default
process.once('exit', close.bind(this));
```

#### 工作进程

worker.js 子进程处理逻辑如下：

- 创建一个 server 对象，注意这里最开始并没有监听 3000 端口
- 通过 message 事件接收主进程 send 方法发送的消息
- 监听 uncaughtException 事件，捕获未处理的异常，发送自杀信息由主进程重建进程，子进程在链接关闭之后退出

```js
const http = require('http');

const server = http.createServer((req, res) => {
  res.writeHead(200, {
    'Content-Type': 'text/plan',
  });
  res.end(`I am worker, pid: ${process.pid}, ppid: ${process.ppid}`);
  throw new Error('worker process exception!'); // 测试异常进程退出、重建
});

let worker;
process.title = 'worker-node';
process.on('message', (message, sendHandle) => {
  if (message === 'server') {
    worker = sendHandle;
    worker.on('connection', (socket) => {
      server.emit('connection', socket);
    });
  }
});

process.on('uncaughtException', (err) => {
  console.log(err);
  process.send({ act: 'suicide' });
  worker.close(() => {
    process.exit(1);
  });
});
```

#### 执行结果

创建了 5 个进程, 一个 master 和 4 个 worker

```verilog
worker process created, pid: 57612 ppid: 57611
worker process created, pid: 57613 ppid: 57611
worker process created, pid: 57614 ppid: 57611
worker process created, pid: 57615 ppid: 57611
```

#### 这里有 2 个问题

##### 问题一: 多个进程都需要监听同一个端口, 怎么实现的?

只有主进程监听了端口, 其他进程只是开启了 HTTP 服务, 没有监听端口, 这完全是可以的

##### 问题二: master 进程怎么将用户请求传递给 worker 进程?

当 master 进程接收到用户请求后会接受到一个`connection`事件, 这时候 master 进程会触发 worker 的`connection`事件,

推荐阅读:

- [源码解析 Node.js 中 cluster 模块的主要功能实现](https://cnodejs.org/topic/56e84480833b7c8a0492e20c)

### 守护进程

守护进程运行在后台不受终端的影响. 守护的意思是说可以守护 web 服务进程不被中断, 如果被中断会再启动一个进程替代

- 创建子进程
- 在子进程中创建新会话（调用系统函数 setsid）
- 改变子进程工作目录（如：“/” 或 “/usr/ 等）
- 父进程终止

```js
// index.js
const { spawn } = require('child_process');

const startDaemon = () => {
  const daemon = spawn('node', ['./daemon.js'], {
    cwd: './',
    detached: true,
    stdio: 'ignore',
  });
  console.log(
    '守护进程开启 父进程 pid: %s, 守护进程 pid: %s',
    process.pid,
    daemon.pid
  );
  daemon.unref();
};

startDaemon();
```

开启一个子进程, 每 10 秒写入一条日志

```js
// daemon.js
const fs = require('fs');
const { Console } = require('console');

// custom simple logger
const logger = new Console(
  fs.createWriteStream('./stdout.log'),
  fs.createWriteStream('./stderr.log')
);

setInterval(() => {
  logger.log('daemon pid: ', process.pid, ', ppid: ', process.ppid);
}, 1000 * 10);
```

#### 测试结果:

1. 主进程执行完会退出
2. 子进程还在执行, 每 10 秒写入一条日志

