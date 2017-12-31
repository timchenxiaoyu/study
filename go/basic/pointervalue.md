go的值函数和引用函数的本质是这样的。如果定义方式
```go
func (p *Sometype) Somemethod (firstArg int) {} 
```
本质是：
```go
func SometypeSomemethod(p *Sometype, firstArg int) {}

```
这个
```go
package main

import (
"fmt"
)

type person struct {
	name string
}

func (p *person)say()  {
	fmt.Println(p.name)
}


func main() {
	var p person
	p.say()

}
```
无论是(p *person)say()还是(p person)say()，方法的调用是没有问题因为p在
go语言中会自动完成初始化，只不过在调用指针方法的时候，做了一次隐式转化罢了。
如果是var p *person，系统任然是回去初始，只不过这时候初始化的值是nil，那么
调用值方法肯定是不行的，因为nil没法转化成空的struct，但是调用指针方法，传入nil
是可以执行，只是这个方法里面不能访问nil的属性罢了
如果把方法改成
```go
func (p *person)say()  {
	fmt.Println(111)
}
```
此时使用var p *person调用(p *person)say() 是没有问题的，当然(p person)say() 
这个肯定是不行的，还是上面说的nil没法转化成空的struct。