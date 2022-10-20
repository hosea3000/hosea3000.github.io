---
title: "golang中的枚举类型"
date: 2022-10-18T22:47:41+08:00
draft: false

categories: ["golang"]
---

⚠️ golang 没有枚举类型

但是我们可以通过类型别名实现一个简单版本的 enmu

```go
package main

import "fmt"

type LogLevel int

const (
    INFO LogLevel = iota
    WARNING
    ERROR
)

func (l LogLevel) String() string {
    s := []string{"INFO", "WARNING", "ERROR"}
    return s[l]
}

func test(level LogLevel) {
    fmt.Println(level)
}

func main() {
    test(ERROR)
}
```

我们可以为类型别名实现 `String()` 打印的时候 golang 会默认调用，所以打印出来是我们要的字符串值。

这里只是使用了类型别名。但是实际上还是 int 类型， 方法调用者还是能传其它的 int 值进来。 暂时没有好的解决办法

不过已经很好了，从方法的签名反应出类型的枚举值

### 使用 go:generate 自动生成 string 方法

如果每次都要自己写类型的string实现很容易出错，也很麻烦。 官方提供了 stringer 专门用来做这个事情

```go
package main

import "fmt"

type Pill int

const (
   Placebo Pill = iota
   Aspirin
   Ibuprofen
   Paracetamol
)

//go:generate stringer -type=Pill
func main() {
   fmt.Println("Hello, world.", Aspirin)
}
```

执行命令 `go generate`会自动为你生成 pill_string.go 实现 string() 方法

### 为什么 go 没有 enum 类型？

golang 的数据类型都有零值。 假如我们写了一个枚举类型的值是 [1，2，3，4，5] 那么他的零值是多少呢？

