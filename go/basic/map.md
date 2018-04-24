## map

```go
	n := make(map[string]int)
	_ ,ok := n["abc"]
	fmt.Println(ok)

	n["abc"]++
	fmt.Println(n["abc"])
	_ ,ok = n["abc"]
	fmt.Println(ok)
	
```

输出

```go
false
1
true
```