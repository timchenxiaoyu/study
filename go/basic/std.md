golang里面有对标准输入和输出的操作方式
```go
	Stdin  = NewFile(uintptr(syscall.Stdin), "/dev/stdin")
	Stdout = NewFile(uintptr(syscall.Stdout), "/dev/stdout")
	Stderr = NewFile(uintptr(syscall.Stderr), "/dev/stderr")
```
第一个需要理解的是，os.Stdout是什么？
它是指针，它是golang将自己的输出指向的指针，程序的标准输出会导入到os.Stdout里面，同理os.Stdin也就是标准输入
先看下面这个例子
```go
package main

import (
	"bytes"
	"io"
	"os"
	"fmt"
)

// not thread safe
func captureStdout(f func()) string {
	fmt.Printf("before os.Stdout :%v \n", os.Stdout)
	old := os.Stdout
	r, w, _ := os.Pipe()
	os.Stdout = w
	fmt.Printf("in w :%v \n", w)
	fmt.Printf("in os.Stdout :%v \n", os.Stdout)
	f()

	w.Close()
	os.Stdout = old
	fmt.Printf("after os.Stdout :%v \n", os.Stdout)
	var buf bytes.Buffer
	io.Copy(&buf, r)
	return buf.String()
}

func doSomething() {
	fmt.Println("This goes to STDOUT")
}

func main() {

	// invoke doSomething and return whatever it writes to STDOUT
	message := captureStdout(doSomething)

	fmt.Printf("get function result is :%s \n", message)

}
```
这是一个截获程序标准输出的例子，先看结果
```go
before os.Stdout :&{0xc42000a140} 
after os.Stdout :&{0xc42000a140} 
get function result is :in w :&{0xc42000a1c0} 
in os.Stdout :&{0xc42000a1c0} 
This goes to STDOUT
```
这个里面先是os.Stdout = w，将w赋值给标准输出，此时标准输出将不会指向/dev/stdout，所以程序的输出会直接导入的w里面
通过io.Copy(&buf, r)获取pipe数据并发到buf里面，然后返回，这样就可以截获输出了。
那如果我们也使用这个标准的输入输出是否可以呢？
答案是肯定的
```go
package main

import (
	"os/exec"
	"os"
	"io"
)

func main() {
	cmd := exec.Command("ls", "-l")
	cmd.Stdout = os.Stdout
	cmd.Stderr = os.Stderr
	cmd.Stdin = os.Stdin
	cmd.Run()
	//os.Stdout = nil
	io.WriteString(os.Stdout, "Hello World \n")
}
```
如果只是设置os.Stdout = nil，那么只有io.WriteString(os.Stdout, "Hello World \n")会受到影响

