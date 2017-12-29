结构体是go实现面向对象编程的拥有熟悉和方法的集合，通过struct去标识

```go
//定义人类
type Human struct {
	name string
	age int
	phone string
}

//定义学生，继承了人类的方法
type Student struct {
	Human //an anonymous field of type Human
	school string
	loan float32
}

//定义雇员，继承了人类
type Employee struct {
	Human //an anonymous field of type Human
	company string
	money float32
}

//人类有说话的方法
func (h *Human) SayHi() {
	fmt.Printf("Hi, I am human %s you can call me on %s\n", h.name, h.phone)
}

// 人类有唱歌的方法
func (h *Human) Sing(lyrics string) {
	fmt.Println("hunamn La la, la la la, la la la la la...", lyrics)
}

// 人类还能喝酒
func (h *Human) Guzzle(beerStein string) {
	fmt.Println("hunamn Guzzle Guzzle Guzzle...", beerStein)
}

// 雇员也有说话的功能，但与普通人说话的内容是不一样的，重写了SayHi
func (e *Employee) SayHi() {
	fmt.Printf("Hi, I am employee %s, I work at %s. Call me on %s\n", e.name,
		e.company, e.phone) //Yes you can split into 2 lines here.
}

// 学生它有没有收入，只能借钱
func (s *Student) BorrowMoney(amount float32) {
	s.loan += amount // (again and again and...)
}

//雇员是可以花钱的，他又收入
func (e *Employee) SpendSalary(amount float32) {
	e.money -= amount // More vodka please!!! Get me through the day!
}

func main() {
	mike := Student{Human{"Mike", 25, "222-222-XXX"}, "MIT", 0.00}
	paul := Student{Human{"Paul", 26, "111-222-XXX"}, "Harvard", 100}
	sam := Employee{Human{"Sam", 36, "444-222-XXX"}, "Golang Inc.", 1000}
	tom := Employee{Human{"Sam", 36, "444-222-XXX"}, "Things Ltd.", 5000}



	fmt.Println(mike.name)
	fmt.Println(paul.loan)
	fmt.Println(sam.money)
	fmt.Println(tom.name)
	
	mike.SayHi()
    tom.SayHi()
}
```


输出
```go
Mike
100
1000
Sam
Hi, I am human Mike you can call me on 222-222-XXX
Hi, I am employee Sam, I work at Things Ltd.. Call me on 444-222-XXX
```
结构体可以随意的组合，这样类似面向对象的继承，如果有相同方法会被重写，通过.就可以直接访问属性和方法