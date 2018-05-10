## map

```go
	n := make(map[string]int)
	aa ,ok := n["abc"]
	fmt.Println(ok,aa)

	n["abc"]++
	fmt.Println(n["abc"])
	_ ,ok = n["abc"]
	fmt.Println(ok)
	
```

输出

```go
false 0
1
true
```
如果不存在会给定一个初始值0，所以下面的++可以直接使用。


## sync map
```go
package main

import (
	"sync"
	"fmt"
)

func main() {

	n := make(map[string]int)
	aa ,ok := n["abc"]
	fmt.Println(ok,aa)

	n["abc"]++
	fmt.Println(n["abc"])
	_ ,ok = n["abc"]
	fmt.Println(ok)

	list := map[string]interface{}{
		"name":          "田馥甄",
		"birthday":      "1983年3月30日",
		"age":           34,
		"hobby":         []string{"听音乐", "看电影", "电视", "和姐妹一起讨论私人话题"},
		"constellation": "白羊座",
		"tag":            34,
	}

	var m sync.Map
	for k, v := range list {
		m.Store(k, v)
	}

	var wg sync.WaitGroup
	wg.Add(2)
	go func() {
		m.Store("age", 22)
		actual, ok := m.LoadOrStore("tag", 8888)
		fmt.Println(actual,ok)
		wg.Done()
	}()

	go func() {
		m.Delete("constellation")
		m.Store("age", 18)
		wg.Done()
	}()

	wg.Wait()

	m.Range(func(key, value interface{}) bool {
		fmt.Println(key, value)
		return true
	})
}
```
通过Store和LoadOrStore装载数据，通过Delete删除数据，通过Range遍历数据