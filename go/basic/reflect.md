## 类型转化
```go
package main

import (
	"fmt"
	"reflect"
)

func main() {
	var x float64 = 3.4
	v := reflect.ValueOf(x)
	fmt.Println("type:", v.Type())
	fmt.Println("kind is float64:", v.Kind() == reflect.Float64)
	fmt.Println("value:", v.Float())

	fmt.Println("----------------------------------")
	var y uint8 = 'x'
	vy := reflect.ValueOf(y)
	fmt.Println("type:", vy.Type())
	fmt.Println("kind is uint8: ", vy.Kind() == reflect.Uint8)
	fmt.Println("value:", vy.Uint())

	fmt.Println("----------------------------------")
	type MyInt int
	var m MyInt = 7
	vz := reflect.ValueOf(m)
	fmt.Println("type:", vz.Type())
	fmt.Println("kind is uint8: ", vz.Kind() == reflect.Int)
	fmt.Println("value:", vz.Int())
}
```
输出的结果如下：
```go
type: float64
kind is float64: true
value: 3.4
----------------------------------
type: uint8
kind is uint8:  true
value: 120
----------------------------------
type: main.MyInt
kind is uint8:  true
value: 7
```

反射遍历struct
```go
type T struct {
    A int
    B string
}
t := T{23, "skidoo"}
s := reflect.ValueOf(&t).Elem()
typeOfT := s.Type()
for i := 0; i < s.NumField(); i++ {
    f := s.Field(i)
    fmt.Printf("%d: %s %s = %v\n", i,
        typeOfT.Field(i).Name, f.Type(), f.Interface())
}
```
输出结果如下
```go
0: A int = 23
1: B string = skidoo
```