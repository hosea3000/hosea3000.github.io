# 单例模式


单例模式就是为了保证一个类全局只有一个实例。且能够被外部使用。

### Golang 实现

#### 利用 lock 实现

```go
package single

import "sync"

var lock = sync.Mutex{}

type single struct {}

var singles *single

func NewSingle() *single {
   lock.Lock()
   defer lock.Unlock()

   if singles == nil {
      singles = &single{}
   }
   return singles
}
```

#### 利用 Sync.Once 实现

```go
package single

import "sync"

type single2 struct {}

var single2Instance *single2

func NewSingle2() *single2 {
	new(sync.Once).Do(func() {
		single2Instance = &single2{}
	})

	return single2Instance
}
```


