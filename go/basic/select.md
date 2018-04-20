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