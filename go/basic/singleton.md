通常的单例模式是通过加锁完成，大概如下：
```go
    mu.Lock()
    defer mu.Unlock()

    if instance == nil {
        instance = &singleton{}    
    }
    return instance
```
但go提供了一个sync的包，它的Once如果Do方法被调用多次，只会运行第一次，
```go
package main

import (
"sync"
"fmt"
)

type singleton struct {
}

var instance *singleton
var once sync.Once

func GetInstance() *singleton {
once.Do(func() {
instance = &singleton{}
})
return instance
}

func main() {
	a := GetInstance()
	b := GetInstance()
	fmt.Println(a == b)
}
```
测试结果a等于b
那它是怎么实现的呢？
```go
func (o *Once) Do(f func()) {
	if atomic.LoadUint32(&o.done) == 1 {
		return
	}
	// Slow-path.
	o.m.Lock()
	defer o.m.Unlock()
	if o.done == 0 {
		defer atomic.StoreUint32(&o.done, 1)
		f()
	}
}
```
有两个重点：第一是它必须等待方法完成才释放锁，这样就保证第一个方法的执行，并且在执行的过程中不会被调用第二次
第二是通过atomic原子操作，去判断方法是否已经被执行，这样以后的方法调用直接return。
细心的可能会发现为啥上面的o.done没有使用atomic.LoadUint32(&o.done)呢？因为外层已经加过锁了，这里就没有必要加锁了