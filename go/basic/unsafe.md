## unsafe

**uintptr是整型,可以足够保存指针的值得范围,在32平台下为4字节,在64位平台下是8字节**


unsafe他是一个可以像c语言那样操作指针的东西
先看一个例子。
```go
package main

import (
	"fmt"
)

func main() {
	var n int32 =2
	var m int64 =3

	n = int32(m)
	fmt.Println(n)
}

```
普通的类型转化，由于降低了精度所以需要注意，但编译器不会有任何报错，程序可以执行,譬如下面
```go
package main

import (
	"fmt"
)

func main() {
	var n int32 =2
	var m int64 =0xFFFFFFFFFFFFFF

	n = int32(m)
	fmt.Println(n)
}

```
只是执行结果是：-1
但程序不会有任何问题。
但如果是指针的强转则会出现问题。
```go
*int32(&n)
```
这样的操作是不允许的，因为指针的操作已经越界了，golang不允许这样访问，那如果你还是想这样玩怎么办
就是上面说的unsafe包，来完成。
```go
package main

import (
	"fmt"
	"unsafe"
)

func main() {
	var n int32 =2
	var m int64 =3

	p := (*int64)(unsafe.Pointer(&n))
	*p = m
	fmt.Println(n)
}

```
输出的结果是：3
这样就可以完成数据的操作了。

看下面这个数组，这个例子更有趣
```go
func main() {

	var m [2]int8

	p := (*int)(unsafe.Pointer(&m[0]))
	*p = 0x1010
	fmt.Println(m)

}
```
输出的结果是：[16 16]
如果m是int16的数组，结果是：[4112 0]
这样就完成了数组的赋值。

还可以深入做一个切片函数：
```go

type SliceHeader struct {
	Data unsafe.Pointer
	Len  int64
	Cap  int64
}

type StringHeader struct {
	Data unsafe.Pointer
	Len  int64
}

func slice(s []int, b, l int64)([]int){
	p := (*SliceHeader)(unsafe.Pointer(&s))
	p.Len = l
	p.Cap = int64(cap(s))
	p.Data = unsafe.Pointer(&s[b])
	return *(*[]int)(unsafe.Pointer(p))
}

func slicestr(s string,b ,l int) (string){
	p := (*StringHeader)(unsafe.Pointer(&s))
	p.Data = unsafe.Pointer(uintptr(p.Data) + uintptr(b))
	p.Len = int64(l)
	return *(*string)(unsafe.Pointer(p))
}


func main() {

	var m [5]int
	s :=m[1: 3]

	fmt.Println(len(s))
	fmt.Println(cap(s))
	fmt.Println(s)
	s = slice(s,0,1)
	fmt.Println(len(s))
	fmt.Println(cap(s))
	fmt.Println(s)

	str := "hello"
	newstr := slicestr(str,1,7)
	fmt.Println(newstr)

}
```
输出的结果是：
```go
2
4
[0 0]
1
4
[0]
elloint1
```
上面最后一张的输出需要注意，建议将len设置成字符串的长度，否则会读取到不应该读取的内容。

最后还可以实现字符串的0拷贝,这个是比较有意思的
```go
func main() {

	 m := []byte{'h','e','l','l','o'}
	 fmt.Println(string(m))
	 p := SliceHeader{
	 	Data: unsafe.Pointer(&m[0]),
	 	Len: int64(len(m)),
	 }

	 str := *(*string)(unsafe.Pointer(&p))
	fmt.Println(str)

}
```
当然不建议这么使用，比较危险，因为gc回收m。可能导致出错





