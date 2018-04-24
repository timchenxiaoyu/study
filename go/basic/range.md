## range

range遍历是一种复制模式
```go
package main

import (
    "fmt"
    "strconv"
)
func main() {
    person:=new(Person)
    persons:=make([]Person,4)
    for i:=0;i<4;i++{
        person.Id= strconv.Itoa(i)
        person.Name =  "sun"
        person.AvatarUrl="AAAAA"
        persons[i]=*person
    }
    fmt.Println("----原始数组-----")
    fmt.Println(persons)
    for _,v :=range persons {
        v.Name="SSS"
    }
    fmt.Println("----range-----")
    fmt.Println(persons)

    for j:=0;j<4 ;j++  {
        persons[j].Name="JJJ"
    }
    fmt.Println("----for-----")
    fmt.Println(persons)
}

type Person struct {
    Id string
    Name string
    AvatarUrl string
}
```
由于range是复制元素，所以没有改变原生数组
```go
----原始数组-----
[{0 sun AAAAA} {1 sun AAAAA} {2 sun AAAAA} {3 sun AAAAA}]
----range-----
[{0 sun AAAAA} {1 sun AAAAA} {2 sun AAAAA} {3 sun AAAAA}]
----for-----
[{0 JJJ AAAAA} {1 JJJ AAAAA} {2 JJJ AAAAA} {3 JJJ AAAAA}]
```
但如果是切片里面是指针，当然可以改变的

```go
package main

import (
	"fmt"
	"strconv"
)
func main() {
	person:=new(Person)
	persons:=make([]*Person,4)
	for i:=0;i<4;i++{
		person.Id= strconv.Itoa(i)
		person.Name =  "sun"
		person.AvatarUrl="AAAAA"
		persons[i]=person
	}
	fmt.Println("----原始数组-----")
	fmt.Println(persons)
	for _, p := range persons {
		fmt.Printf("%#v\n", p)
	}
	for _,v :=range persons {
		v.Name="SSS"
	}
	fmt.Println("----range-----")
	fmt.Println(persons)
	for _, p := range persons {
		fmt.Printf("%#v\n", p)
	}

	for j:=0;j<4 ;j++  {
		persons[j].Name="JJJ"
	}
	fmt.Println("----for-----")
	fmt.Println(persons)
	for _, p := range persons {
		fmt.Printf("%#v\n", p)
	}
}

type Person struct {
	Id string
	Name string
	AvatarUrl string
}
```
结果如下：
```go
----原始数组-----
[0xc4200740c0 0xc4200740c0 0xc4200740c0 0xc4200740c0]
&main.Person{Id:"3", Name:"sun", AvatarUrl:"AAAAA"}
&main.Person{Id:"3", Name:"sun", AvatarUrl:"AAAAA"}
&main.Person{Id:"3", Name:"sun", AvatarUrl:"AAAAA"}
&main.Person{Id:"3", Name:"sun", AvatarUrl:"AAAAA"}
----range-----
[0xc4200740c0 0xc4200740c0 0xc4200740c0 0xc4200740c0]
&main.Person{Id:"3", Name:"SSS", AvatarUrl:"AAAAA"}
&main.Person{Id:"3", Name:"SSS", AvatarUrl:"AAAAA"}
&main.Person{Id:"3", Name:"SSS", AvatarUrl:"AAAAA"}
&main.Person{Id:"3", Name:"SSS", AvatarUrl:"AAAAA"}
----for-----
[0xc4200740c0 0xc4200740c0 0xc4200740c0 0xc4200740c0]
&main.Person{Id:"3", Name:"JJJ", AvatarUrl:"AAAAA"}
&main.Person{Id:"3", Name:"JJJ", AvatarUrl:"AAAAA"}
&main.Person{Id:"3", Name:"JJJ", AvatarUrl:"AAAAA"}
&main.Person{Id:"3", Name:"JJJ", AvatarUrl:"AAAAA"}
```
此时就可以改变了。