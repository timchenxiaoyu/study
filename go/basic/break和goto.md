##  break 

通常的break只是跳出本次循环和contine loop效果一样，如下
```go
func main() {
	outer:
	for i := 0; i < 3; i++ {
		for j := 0; j < 3; j++ {
			print(i, ",", j, " ")
			continue outer
		}
		println()
	}
}
```
输出
```go
0,0 1,0 2,0 
```
可以看到每次执行完print后程序跳出到outer地方执行下一次外层循环，甚至连println都没有执行，这点需要注意

如果是break
```go
func main() {
	//outer:
	for i := 0; i < 3; i++ {
		for j := 0; j < 3; j++ {
			print(i, ",", j, " ")
			//continue outer
			break
		}
		println()
	}
}
```
break 只是跳出了内层循环，所以println方法会得到执行，输出如下：
```go
0,0 
1,0 
2,0 
```
那如果想结束整个循环呢，只需要break outer就可以了
```go
func main() {
	outer:
	for i := 0; i < 3; i++ {
		for j := 0; j < 3; j++ {
			print(i, ",", j, " ")
			//continue outer
			break outer
		}
		println()
	}
}
```
结果如下：
```go
0,0 
```

## goto
goto则是任意的跳转，继续执行
```go
func main() {
	Loop:
	fmt.Println("test")
	for a:=0;a<5;a++{
		fmt.Println(a)
		if a>3{
			goto Loop
		}
	}
}
```
程序会死循环下去
```go
test
0
1
2
3
4
test
0
1
2
3
4
...
```
goto就是调到指定的地方继续运行
如果想结束只需要吧label放到下面即可如下：
```go
func main() {
	for a:=0;a<5;a++{
		fmt.Println(a)
		if a>3{
			goto Loop
		}
	}
	Loop:           
	fmt.Println("test")
}
```
那么他就只会运行一次了，结果如下
```go
0
1
2
3
4
test
```