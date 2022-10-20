# V8 的内存限制与垃圾回收


# V8 的内存限制与垃圾回收

### V8 的内存限制

node 使用 V8 作为 javaScript 脚本引擎

#### v8 的内存限制和对象分配

- v8 中所有 javascript 对象都是通过堆内存进行分配的
- 内存限制：64 位系统大约 1.4G，32 位系统大约 0.7G, 当总使用内存超过 1.4G 之后就会 OOM(Out Of Memory)。进程会直接退出

#### 为何要内存限制

- 表层原因为 v8 最初为浏览器设计，不太可能遇到大量的内存的场景。对于网页来说，v8 的限制已经绰绰有余
- 深层原因是 v8 的垃圾回收机制的限制.
  1.4G V8 进行一次垃圾回收需要的时间大约是 50ms, 阻塞主线程

#### 自定义内存限制

这两个值必须启动的时候指定, 启动后无法修改, 所以 V8 内存无法根据情况自动扩容

```shell
node --max-old-space-size=1700 test.js // 单位为MB
node --max-new-space-size=1024 test.js // 单位为KB
```

#### 查看内存

```js
process.memoryUsage()
{
  rss: 23863296, // rss是Resident Set Size的缩写，为常驻内存的总大小
  heapTotal: 6057984, // 已经申请到的堆内存
  heapUsed: 2523608, // 当前使用的堆内存
  external: 1270913 // 堆外内存
}

// 查看操作系统总内存
os.totalmem()

// 查看操作系统的空闲内存
os.freemem()
```

### 垃圾回收

### 新生代和老生代

V8 将内存分为两类：新生代内存空间和老生代内存空间，新生代内存空间主要用来存放存活时间较短的对象，老生代内存空间主要用来存放存活时间较长的对象。对于垃圾回收，新生代和老生代有各自不同的策略

### 新生代的垃圾回收(Scavenge)

新生代需要清理的存活对象较少, 所以选择复制存活对象的算法

具体实现时主要采用了 Cheney 算法。Cheney 将内存空间一分为二， 一个处于使用，一个处于闲置。处于使用中的也叫作 From，处于闲置中的也叫作 To。垃圾回收的过程就是将 From 中存活的对象复制到 To 中

1. 在垃圾回收的过程中，如果发现某个对象之前被清理过，那么会将其晋升到老生代内存空间中
2. 当要从 From 复制一个对象到 To 空间时，如果 To 空间中的使用量已经超过了 25%，那么就将 From 中的对象直接晋升到老生代内存空间中
3. 缺点是只是用了一半的内存, 优点是速度快

### 老生代的垃圾回收(Mark Sweep & Mark Compact)

老生代中存活的对象较多, 需要清理的对象较少, 所以选择清楚死亡对象的算法

#### Mark Sweep(标记清除)

分为标记和清除两个阶段, 在标记阶段标记所有存活的对象, 在清除阶段清除所有标记的对象
缺点: 会产生内存碎片

#### Mark Compact(标记整理)

在整理的过程中将存活的对象往一端移动, 移动完成后, 直接清理掉边界内存
缺点: 速度慢

#### 什么时候执行 GC

1. 定时 GC
2. 内存不够分配时
3. 手动 GC

#### 总结

V8 主要是 Mark Sweep 算法回收内存, 在空间不足以分配给新生代晋升的对象时才会采用 Mark Compact 方式进行垃圾回收

#### 查看垃圾回收日志

```js
// $ node --trace_gc

// 全局作用域的GC
arr = [];
for (let i = 0; i < 3000 * 10000; i++) {
  arr.push(i + 'test');
}
arr = undefined;
process.memoryUsage();

// 方法作用域的垃圾回收(连续调用两次)
function test(num) {
  let arr = [];
  for (let i = 0; i < 10000 * num; i++) {
    arr.push(i + 'test');
  }
  arr = undefined;
}
process.memoryUsage();
```

#### 手动进行垃圾回收

```js
// $ node --expose-gc
global.gc();
```

### 内存使用

#### 1. 作用域

将变量声明在作用域内有利于垃圾回收, 避免使用全局变量

#### 2. 闭包

闭包会导致垃圾回收失效, 尽量减少使用

#### 3. 使用堆外内存

将对象以 Buffer 的形式保存, Buffer 不受 V8 内存限制

#### 4. 使用 Stream

使用 Stream 避免一次读取过多的数据到内存(要使用 pipe)

### 内存泄露排查工具

- [heapdump+chrome devTools](https://www.ctolib.com/topics-118921.html)
- [easy-monitor](https://www.jianshu.com/p/791a9ba77abb)

以上内容主要来自<深入浅出 Node.js> 第五章

### 内存打印工具

```js
/**
 * 封装 print 方法输出内存占用信息
 */
const print = function () {
  const memoryUsage = process.memoryUsage();

  /**
   * 单位为字节格式为 MB 输出
   */
  const format = function (bytes) {
    return (bytes / 1024 / 1024).toFixed(2) + ' MB';
  };

  console.log(
    JSON.stringify({
      rss: format(memoryUsage.rss),
      heapTotal: format(memoryUsage.heapTotal),
      heapUsed: format(memoryUsage.heapUsed),
      external: format(memoryUsage.external),
    })
  );
};
```

```js

test(){
  const arr ==[]
  return ()=>{
    arr.push(1)
  }
}

const fun = test()

fun()

```

