# 如何在 Mac OS 下用 node 调用 C


## 如何在 Mac OS 下用 node 调用 C

> [如何调用 C 代码](https://www.cnblogs.com/binchage/p/11043973.html)
>
> [如何在 Mac os 编译 C 文件](https://blog.csdn.net/ssihc0/article/details/17299381)

> 使用 node-ffi 可以让 Node.js 调用 C++ 的 Library 。在 Windows 下是 `dll` ，在 Mac OS 下是 `dylib` ，Linux 则是 `so` 。node-ffi 加载 Library 是有限制的，只能处理 C 风格的 Library 。也就是函数要被放在 `extern "C"` 里。

### 如何在 Mac OSX 中制作 dylib 和使用 dylib

1.首先是构建一个函数库

编辑 add.c

```c
int add(int a,int b) {
  return a+b;
}

int axb(int a,int b) {
	return a*b;
}
```

2.编译函数库得到 libadd.dylib

```shell
gcc -c add.c -o add.o

//下面是linux系统时
ar rcs libadd.a add.o

//下面是Mac OSX
gcc add.o -dynamiclib -current_version 1.0  -o libadd.dylib
```

### node-ffi 调用 C 编译的文件

```javascript
var ref = require('ref');
var ffi = require('ffi');

var intPtr = ref.refType(ref.types.int); // 创建一个 int 指针类型

var lib = ffi.Library('libadd', {
  add: ['int', ['int', 'int']],
  axb: ['int', ['int', 'int']],
});

let sum = lib.add(1, 2);
console.log(`1 + 2 = ${sum}`);

let res = lib.axb(3, 3);
console.log(`3 * 3 = ${res}`);
```

