通过管道可以实现多个进程间通信，协程之间也是可以的
```go
package main

import (
	"fmt"
	"time"
	"os"
	"io/ioutil"
)

func NewPipe() (*os.File, *os.File, error) {
	read, write, err := os.Pipe()
	if err != nil {
		return nil, nil, err
	}
	return read, write, nil
}

func main() {

	fmt.Println("main starting")
	stop := make(chan int)
	r ,w ,_:= NewPipe()
	go func() {
		fmt.Println("int go run")
		msg,_ := ioutil.ReadAll(r)
		fmt.Printf("int go run get msg %s \n",string(msg))
		<-stop
	}()

	w.WriteString("hello world")
	w.Close()
	time.Sleep(time.Second*3)
	stop <- 1
	fmt.Println("main end")

}
```
上面的代码先创建一个管道，分别给main和协程，运行并测试
```go
main starting
int go run
int go run get msg hello world 
main end
```