在for循环里面使用goroutine闭包需要注意
先看一个例子热身
```go
package main

import (
	"sync"
	"fmt"
)

const N = 10

func main() {
	m := make(map[int]int)

	wg := &sync.WaitGroup{}
	mu := &sync.Mutex{}
	wg.Add(N)
	for i := 0; i < N; i++ {
		go func() {
			defer wg.Done()
			mu.Lock()
			fmt.Println(i)
			m[i] = i
			mu.Unlock()
		}()
	}
	wg.Wait()
	fmt.Println(m)
}
```
输出的结果如下
```go
10
10
10
10
10
10
10
10
10
10
map[10:10]
```
这里可以得出结论就是闭包可以直接读取外层的变量，并且goroutine和外层的for循环是同时执行的，

那么先让他睡眠一下看看
```go
import (
	"sync"
	"time"
	"fmt"
)

const N = 10

func main() {
	m := make(map[int]int)

	wg := &sync.WaitGroup{}
	mu := &sync.Mutex{}
	wg.Add(N)
	for i := 0; i < N; i++ {
		go func() {
			defer wg.Done()
			mu.Lock()
			fmt.Println(i)
			m[i] = i
			mu.Unlock()
		}()
		time.Sleep(1 * time.Second)
	}
	wg.Wait()
	fmt.Println(m)
}
```
我们给每个goroutine运行的时间，输出的结果如下
```go
0
1
2
3
4
5
6
7
8
9
map[1:1 2:2 3:3 6:6 7:7 8:8 9:9 0:0 4:4 5:5]
```
那么难道每次都要这样写，当然不是，所以我们需要把每次变量传入到闭包内部,代码如下
```go
import (
	"sync"

	"fmt"
)

const N = 10

func main() {
	m := make(map[int]int)

	wg := &sync.WaitGroup{}
	mu := &sync.Mutex{}
	wg.Add(N)
	for i := 0; i < N; i++ {
		go func(i int) {
			defer wg.Done()
			mu.Lock()
			fmt.Println(i)
			m[i] = i
			mu.Unlock()
		}(i)

	}
	wg.Wait()
	fmt.Println(m)
}

```
这样就可以得到上面相同的输出了。