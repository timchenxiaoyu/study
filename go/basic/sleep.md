sleep方法接受一个Duration的参数
```go
package main
import (
	"math/rand"
	"time"
	"fmt"
)

func main() {
	rand.Seed(time.Now().UnixNano())
	a := rand.Intn(10)
	fmt.Println(a)
	time.Sleep(time.Duration(a)*time.Second)
}
```
如果你使用6*time.Second当然也没有问题,
但如果是通过b=a*time.Second这句话有问题，因为类型不匹配无法相乘
```go
func Sleep(d Duration)
```