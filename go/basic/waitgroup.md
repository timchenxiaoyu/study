## waitgroup
入门example
```go
import (
        "sync"
        "fmt"
)
func main() {
	var wg sync.WaitGroup
	for i :=0 ;i<5;i++{
        wg.Add(1)
		go func(i int) {
			defer wg.Done()
			fmt.Println(i)
		}(i)
	}
	wg.Wait()
}
```
**上面的方法，var wg sync.WaitGroup 其实一个坑，需要注意，因为传参是值传递，这会导致问题**
譬如下面代码
```go
go func(text string,ch chan map[string]int, wg sync.WaitGroup){
			defer wg.Done()

			m := make(map[string]int)
			ch <-m
			fmt.Println("abc")

		}(array[i],ch,wg)//值传递wg
```
这就会导致死锁，所以通用的解决办法是
```go
var wg = &sync.WaitGroup{}
```
这样创建的是指针，那么久不存在上面的问题了。函数解释  wg *sync.WaitGroup的传参即可。
