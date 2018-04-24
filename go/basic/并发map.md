如果是在1.9版本之前实现一个并发的map，如下
```go

package main

import (
	"math/rand"
	"sync"
)

const N = 10

var counter = struct{
	m map[int]int
	sync.RWMutex
}{m : make(map[int]int),
}
func main() {

	wg := &sync.WaitGroup{}
	wg.Add(N)
	for i := 0; i < N; i++ {
		go func() {
			defer wg.Done()
			counter.Lock()
			counter.m[rand.Int()] = rand.Int()
			counter.Unlock()
		}()
	}
	wg.Wait()
	println(len(counter.m))
}
```
输出的结果是10。如果去掉锁，会出现panic并发写map的异常。

**带mutex的struct必须是指针receivers**