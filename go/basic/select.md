## select多路复用

```go
func main() {
	//fmt.Println("Commencing countdown.")
	//tick := time.Tick(1 * time.Second)
	//for countdown := 10; countdown > 0; countdown-- {
	//	fmt.Println(countdown)
	//	fmt.Println(<-tick)
	//}

	abort := make(chan struct{})
	go func() {
		time.Sleep(time.Second*7)
		abort <- struct{}{}
	}()
	fmt.Println("Commencing countdown.  Press return to abort.")
	tick := time.Tick(2 * time.Second)
	for countdown := 10; countdown > 0; countdown-- {
		fmt.Println(countdown)
		select {
		case <-tick:
		// Do nothing.
		case <-abort:
			fmt.Println("Launch aborted!")
			return
		case <-time.After(1 * time.Second):
			fmt.Println("timeout!")
			return
		}
	}
}

```
time.After是在单次循环里面时间
而上面的abort是for循环总时间的

## 停止协程
通过chan+select就可以优雅停止协程
```go
func main() {
	stop := make(chan bool)
	go func() {
		for {
			select {
			case <-stop:
				fmt.Println("监控退出，停止了...")
				return
			default:
				fmt.Println("goroutine监控中...")
				time.Sleep(2 * time.Second)
			}
		}
	}()
	time.Sleep(10 * time.Second)
	fmt.Println("可以了，通知监控停止")
	stop<- true
	//为了检测监控过是否停止，如果没有监控输出，就表示停止了
	time.Sleep(5 * time.Second)
}
```