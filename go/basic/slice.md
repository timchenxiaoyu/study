## 切片追加

数组切片
```go
    s := []int{1, 2, 3}
	ss := s[1:]
	ss[0]=100
```
上面的例子，由于切片是指针，所以当ss改变的后，s的值也会一起改变
但当追加元素过后呢
```go
    s := []int{1, 2, 3}
	ss := s[1:]
	ss = append(ss, 4)
	ss[0]=100
```
此时情况就变了，因为s的len和cap都是3，ss是len和cap分别是2，2 所以当append过后
由于超出cap会重新创建一个切片，所以此后ss的变化都不会影响到s
完整代码如下
```go
package main

import (
	"fmt"
)

func main() {
	s := []int{1, 2, 3}
	fmt.Printf("%p  %p  %p\n",&s,&s[0],&s[1])
	ss := s[1:]
	ss[0]=100
	fmt.Println(len(s),cap(s))
	fmt.Println(len(ss),cap(ss))
	fmt.Printf("%p  %p  %p\n",&ss,&ss[0],&ss[1])
	ss = append(ss, 4)
	fmt.Println(len(s),cap(s))
	fmt.Println(len(ss),cap(ss))

	for _, v := range ss {
		v += 10
	}

	for i := range ss {
		ss[i] += 10
	}

	fmt.Println(s)
	fmt.Println(ss)
}
```
结果如下：
```go
0xc42007c080  0xc42007c0a0  0xc42007c0a8
3 3
2 2
0xc42007c0e0  0xc42007c0a8  0xc42007c0b0
3 3
3 4
[1 100 3]
[110 13 14]
```

如果两个切片追加可以通过
```go
append([]int{1, 2}, []int{3, 4}...)
```

## 修改切片
```go
func modify(s []int) {
    s[0] = 0
}
func main() {
    s := []int{1, 2, 3}
    modify(s)
    fmt.Println(s)
}

```
上面的方法肯定能修改的，没有问题，但下面的情况则不一样
```go
package main

import (
	"fmt"
)
func pop(s []int) {
	s = s[:len(s)-1]
}
func main() {
	s := []int{1, 2, 3}
	pop(s)
	fmt.Println(s)
}
```
输出的结果是
```go
[1 2 3]
```
上面s的值并没有变化，是因为pop里面的s是值传递，s在pop内部已经被重新赋值了。当然修改还是会成功的
```go
func pop(s []int) {
	s = s[:len(s)-1]
	s[1]=100
}
```
那么s将会是
```go
[1 100 3]
```


## 切片排序
```go
package main

import (
	"fmt"
	"sort"
)

type S struct {
	v int
}

func main() {
	s := []S{{1}, {3}, {5}, {2}}
	sort.Slice(s, func(i, j int) bool { return s[i].v < s[j].v })
	fmt.Printf("%#v", s)
}
```