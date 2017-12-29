接口就是一组方法签名的集合
```go
package main
import "fmt"

type Human struct {
	name string
	age int
	phone string
}

type Student struct {
	Human //an anonymous field of type Human
	school string
	loan float32
}

type Employee struct {
	Human //an anonymous field of type Human
	company string
	money float32
}

//A human method to say hi
func (h Human) SayHi() {
	fmt.Printf("Hi, I am %s you can call me on %s\n", h.name, h.phone)
}

//A human can sing a song
func (h Human) Sing(lyrics string) {
	fmt.Println("La la la la...", lyrics)
}

//Employee's method overrides Human's one
func (e Employee) SayHi() {
	fmt.Printf("Hi, I am %s, I work at %s. Call me on %s\n", e.name,
		e.company, e.phone) //Yes you can split into 2 lines here.
}

// Interface Men is implemented by Human, Student and Employee
// because it contains methods implemented by them.
type Men interface {
	SayHi()
	Sing(lyrics string)
}

func main() {
	mike := Student{Human{"Mike", 25, "222-222-XXX"}, "MIT", 0.00}
	paul := Student{Human{"Paul", 26, "111-222-XXX"}, "Harvard", 100}
	sam := Employee{Human{"Sam", 36, "444-222-XXX"}, "Golang Inc.", 1000}
	Tom := Employee{Human{"Sam", 36, "444-222-XXX"}, "Things Ltd.", 5000}

	//a variable of the interface type Men
	var i Men

	//i can store a Student
	i = mike
	fmt.Println("This is Mike, a Student:")
	i.SayHi()
	i.Sing("November rain")

	//i can store an Employee too
	i = Tom
	fmt.Println("This is Tom, an Employee:")
	i.SayHi()
	i.Sing("Born to be wild")

	//a slice of Men
	fmt.Println("Let's use a slice of Men and see what happens")
	x := make([]Men, 3)
	//These elements are of different types that satisfy the Men interface
	x[0], x[1], x[2] = paul, sam, mike

	for _, value := range x{
		value.SayHi()
	}
}

```
上面的接口Men里面有两个方法，SayHi和Sing，无论是Student还是Employee都实现这个方法，都可以直接赋值给Men
```go
This is Mike, a Student:
Hi, I am Mike you can call me on 222-222-XXX
La la la la... November rain
This is Tom, an Employee:
Hi, I am Sam, I work at Things Ltd.. Call me on 444-222-XXX
La la la la... Born to be wild
Let's use a slice of Men and see what happens
Hi, I am Paul you can call me on 111-222-XXX
Hi, I am Sam, I work at Golang Inc.. Call me on 444-222-XXX
Hi, I am Mike you can call me on 222-222-XXX
```