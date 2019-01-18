## chan

关闭的chan意味着什么呢，就是不能继续写入了，但读取时没有问题的，如果没有会返回默认值，如果需要判断可以通过ok语法
```go
package main

import "fmt"

func main() {
	ch := make(chan string,1)
	ch <- "sbc"
	close(ch)
	//for c := range ch{
	//	fmt.Println(c)
	//}
	a,ok := <-ch
	fmt.Println(a,ok)

	b,ok := <-ch
	fmt.Println(b,ok)

	c,ok := <-ch
	fmt.Println(c,ok)
}

```
输出：
```go
sbc true
 false
 false
```
当然写入报错。

## 长度

```go
    ch := make(chan int ,100)
	ch <-1
	ch<-1
	fmt.Println(len(ch),cap(ch))
```
输出的结果是：
```go
2 100
```
长度是里面元素的个数，cap是容量，只有容量超的时候才会阻塞

下面的代码是有问题，协程是没法停止的
```
package main

import (
	"time"
	"fmt"
	"runtime"
)

func main() {

	ch := make(chan int)
	go func() {
		time.Sleep(time.Second*6)
		ch <-1
	}()


	select {
	case <- ch:
		fmt.Println("over")
	case <- time.After(3*time.Second):
		fmt.Println("timeout")

	}

	for{
		time.Sleep(time.Second *1)
		fmt.Println(runtime.NumGoroutine())
	}

}

func app(s *[]byte){
	*s = append(*s,'d','e','f')
}
```
在程序协程执行完由于管道没有空间会一直阻塞，协程会卡住
需要将代码如下修改
```
	ch := make(chan int,1)
```
这个在编程的时候容易忽视