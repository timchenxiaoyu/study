
自 Go 1.5开始， Go的GOMAXPROCS默认值已经设置为 CPU的核数
如果在一个核上面执行的话，可以通过Gosched主动让出执行权，如下
```go
package main

import (
	"fmt"
	"runtime"
	"sync"
)

const N = 26

func main() {
	const GOMAXPROCS = 1
	runtime.GOMAXPROCS(GOMAXPROCS)

	var wg sync.WaitGroup
	wg.Add(2 * N)
	for i := 0; i < N; i++ {
		go func(i int) {
			defer wg.Done()
			runtime.Gosched()
			fmt.Printf("%c", 'a'+i)
		}(i)
		go func(i int) {
			defer wg.Done()
			fmt.Printf("%c", 'A'+i)
		}(i)
	}
	wg.Wait()
}
```
执行结果如下
```go
ZABCDEFGHIJKLMNOPQRSTUVWXYabcdefghijklmnopqrstuvwxyz
```
这样看到大小写字母是分开显示的，如果是多核的情况下，执行就不可控制了。