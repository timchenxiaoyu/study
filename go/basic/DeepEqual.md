如何比较对象内容是否一样呢，譬如map或者struct。
通过自己编写的比较方法就是遍历属性比较，这个效率比较高，其实golang提供一个通用的反射方式，看下面一个maop比较的例子
```go
func main() {

	m1:=map[string]int{"a":1,"b":2,"c":3}
	m2:=map[string]int{"a":1,"c":3,"b":2}
	fmt.Println("reflect.DeepEqual(m1,m2) = ",reflect.DeepEqual(m1,m2))
}
```
输出结果如下
```go
reflect.DeepEqual(m1,m2) =  true
```
如果是struct也是一样的
```go

import (
"fmt"
"reflect"
)

type S struct {
	a, b, c string
}

func main() {
	x := S{"a", "b", "c"}
	y := S{"a", "b", "c"}
	fmt.Println(reflect.DeepEqual(x, y))
}
```
结果如下
```go
true
```