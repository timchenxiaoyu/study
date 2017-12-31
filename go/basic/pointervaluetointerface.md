结构体组合，如果子结构体有相同的方法，此时方法是会子结构体方法会覆盖父结构体

看下面的例子

```go

package main
import "fmt"

type Human struct {
	name string
	age int
	phone string
}


type Employee struct {
	*Human //an anonymous field of type Human
	company string
	money float32
}

//A human method to say hi
func (h *Human) SayHi() {
	fmt.Printf("Hi, I am a Human %s you can call me on %s\n", h.name, h.phone)
}


//Employee's method overrides Human's one
func (e Employee) SayHi() {
	fmt.Printf("Hi, I am a Employee %s, I work at %s. Call me on %s\n", e.name,
		e.company, e.phone) //Yes you can split into 2 lines here.
}

// Interface Men is implemented by Human, Student and Employee
// because it contains methods implemented by them.
type Men interface {
	SayHi()
}

func main() {
	mike := &Employee{&Human{"Mike", 25, "222-222-XXX"}, "MIT", 0.00}
	mike.SayHi()
}
```
输出的结果为：
```go
Hi, I am a Employee Mike, I work at MIT. Call me on 222-222-XXX
```
其实父结构体和子结构体如果有相同方法，不论是否是值函数还是指针函数，也不管匿名组合的时候是以值的方式还是以指针的方式都一样

**但是如果是把这个结构体赋值给一个接口的时候，此时需要注意**

```go

func main() {
	mike := Employee{Human{"Mike", 25, "222-222-XXX"}, "MIT", 0.00}
	var i Men
	i = mike
	i.SayHi()
}
```
上面的i = mike是一个需要注意的地方，方法调用go是有隐式转化的，所以不论是以值方式还是指针方式都可以，但是如果是
接口赋值则必须对应，譬如
Men接口需要一个SayHi的方法，如果(e Employee) SayHi实现的，那么Employee{Human{"Mike", 25, "222-222-XXX"}, "MIT", 0.00}
这个对象就具有这个方法当然没有任何问题，但如果是(e *Employee) SayHi，那么上面的Employee则不具备这个方法此时则会报错
可以通过&Employee的方式
如果你删除子类的Employee没有这个方法，使用父结构体集成的方法也是可以的，此时有意思的是：
如果父结构体是以引用的方式注入：
mike := Employee{&Human{"Mike", 25, "222-222-XXX"}, "MIT", 0.00}
那么父结构体实现这个方法既可以是值函数也可以是指针函数
如果父结构体是以值的方式注入：
mike := Employee{Human{"Mike", 25, "222-222-XXX"}, "MIT", 0.00}
那么父结构体只能是值函数