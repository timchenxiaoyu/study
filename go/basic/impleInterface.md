sort包里面Sort方法如下
```go
func Sort(data Interface) {
	n := data.Len()
	quickSort(data, 0, n, maxDepth(n))
}
```
需要传入一个Interface,这个Interface如下

```go
type Interface interface {
	// Len is the number of elements in the collection.
	Len() int
	// Less reports whether the element with
	// index i should sort before the element with index j.
	Less(i, j int) bool
	// Swap swaps the elements with indexes i and j.
	Swap(i, j int)
}
```
所以只要实现这三个方法的的对象都可以传入昨晚参数

下面以HumanGroup为例实现上面三个方法
```go
package main
import (
    "fmt"
    "strconv"
    "sort"
)

type Human struct {
    name string
    age int
    phone string
}

func (h Human) String() string {
    return "(name: " + h.name + " - age: "+strconv.Itoa(h.age)+ " years)"
}

type HumanGroup []Human //HumanGroup is a type of slices that contain Humans

func (g HumanGroup) Len() int {
    return len(g)
}

func (g HumanGroup) Less(i, j int) bool {
    if g[i].age < g[j].age {
        return true
    }
    return false
}

func (g HumanGroup) Swap(i, j int){
    g[i], g[j] = g[j], g[i]
}

func main(){
    group := HumanGroup{
        Human{name:"Bart", age:24},
        Human{name:"Bob", age:23},
        Human{name:"Gertrude", age:104},
        Human{name:"Paul", age:44},
        Human{name:"Sam", age:34},
        Human{name:"Jack", age:54},
        Human{name:"Martha", age:74},
        Human{name:"Leo", age:4},
    }

    //Let's print this group as it is
    fmt.Println("The unsorted group is:")
    for _, v := range group{
        fmt.Println(v)
    }

    //Now let's sort it using the sort.Sort function
    sort.Sort(group)

    //Print the sorted group
    fmt.Println("\nThe sorted group is:")
    for _, v := range group{
        fmt.Println(v)
    }
}

```
这样就可以按照年龄大小进行排序了，输出如下：
```go
The unsorted group is:
(name: Bart - age: 24 years)
(name: Bob - age: 23 years)
(name: Gertrude - age: 104 years)
(name: Paul - age: 44 years)
(name: Sam - age: 34 years)
(name: Jack - age: 54 years)
(name: Martha - age: 74 years)
(name: Leo - age: 4 years)

The sorted group is:
(name: Leo - age: 4 years)
(name: Bob - age: 23 years)
(name: Bart - age: 24 years)
(name: Sam - age: 34 years)
(name: Paul - age: 44 years)
(name: Jack - age: 54 years)
(name: Martha - age: 74 years)
(name: Gertrude - age: 104 years)

```