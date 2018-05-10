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
当然写入会报错。

