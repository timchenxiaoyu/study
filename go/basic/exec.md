## exec

exec是golang执行命令的工具包
看一个简单执行的例子
```go
import (
	"fmt"
	"os/exec"
)

func main() {
	cmd := exec.Command("cat", "/Users/chenxy/go/src/workspace/publickey.crt")
	out, err := cmd.Output()
	if err != nil {
		fmt.Errorf("err %v \n",err)
	}
	fmt.Println(string(out))
}
```
下面看使用标准输入输出
```go
func main() {
	// 执行系统命令
	// 第一个参数是命令名称
	// 后面参数可以有多个，命令参数
	cmd := exec.Command("cat", "/Users/chenxy/go/src/workspace/publickey.crt")
	// 获取输出对象，可以从该对象中读取输出结果
	stdout, err := cmd.StdoutPipe()
	if err != nil {
		fmt.Errorf("err %v \n",err)
	}
	// 保证关闭输出流
	defer stdout.Close()
	// 运行命令
	if err := cmd.Start(); err != nil {
		fmt.Errorf("err %v \n",err)
	}
	// 读取输出结果
	opBytes, err := ioutil.ReadAll(stdout)
	if err != nil {
		fmt.Errorf("err %v \n",err)
	}
	fmt.Println(string(opBytes))
}
```
就可以得到cat输出的结果了。
也可以直接输出的到标准输出
```go
package main

import (
	"fmt"
	//"io/ioutil"
	"os/exec"
	"os"
	"io"
)

func main() {
// 执行系统命令
// 第一个参数是命令名称
// 后面参数可以有多个，命令参数
cmd := exec.Command("cat", "/Users/chenxy/go/src/workspace/publickey.crt")
// 获取输出对象，可以从该对象中读取输出结果
stdout, err := cmd.StdoutPipe()

if err != nil {
fmt.Errorf("err %v \n",err)
}
// 保证关闭输出流
defer stdout.Close()
fmt.Println("start")
if err := cmd.Start(); err != nil {
fmt.Errorf("err %v \n",err)
}
io.Copy(os.Stdout,stdout)
fmt.Println("end")
}
```
通过io.Copy(os.Stdout,stdout)直接送到标准输出,Copy是知道src是EOF时候才结束，结束返回err是nil
当然如果你想把结果重定向到文件也是ok的，
````go
package main

import (
	"fmt"
	//"io/ioutil"
	"os/exec"
	"os"
)

func main() {
	cmd := exec.Command("ls", "-a", "-l")
	// 重定向标准输出到文件
	f, err := os.OpenFile("stdout.log", os.O_CREATE|os.O_WRONLY, 0600)
	if err != nil {
		fmt.Errorf("err %v \n",err)
	}
	defer f.Close()
	cmd.Stdout = f
	// 执行命令
	if err := cmd.Start(); err != nil {
		fmt.Errorf("err %v \n",err)
	}
}
````
cmd.Stdout是一个接口，在启动Start方法里面会调用改接口的Write方法，写数据到改文件
上面都是标准输出的使用，如果想使用标准输入麻烦一点，看下面代码
```go
package main

import (
	"fmt"
)

func main() {
	fmt.Println("Hello, What's your favorite number?")
	var i int
	fmt.Scanf("%d\n", &i)
	fmt.Println("Ah I like ", i, " too.")
}

package main

import (
	"fmt"
	"io"
	"os"
	"os/exec"
)

func main() {
	subProcess := exec.Command("go", "run", "testmain/main.go") //Just for testing, replace with your subProcess

	stdin, err := subProcess.StdinPipe()
	if err != nil {
		fmt.Println(err) //replace with logger, or anything you want
	}
	defer stdin.Close() // the doc says subProcess.Wait will close it, but I'm not sure, so I kept this line

	subProcess.Stdout = os.Stdout
	subProcess.Stderr = os.Stderr

	fmt.Println("START") //for debug
	if err = subProcess.Start(); err != nil { //Use start, not run
		fmt.Println("An error occured: ", err) //replace with logger, or anything you want
	}

	io.WriteString(stdin, "4\n")
	subProcess.Wait()
	fmt.Println("END") //for debug
}

```
运行测试一下：
```go
go run main.go
START
Hello, What's your favorite number?
Ah I like  4  too.
END
```
直接运行也是可以的，

还有一个细节，就是Start和Run方法是有区别的
**Start执行不会等待命令完成就，Run会阻塞等待命令完成。**
```go
package main

import (
	"fmt"
	"os/exec"
)

func main() {
	cmd := exec.Command("sleep", "5")
	// 如果用Run，执行到该步则会阻塞等待5秒
	err := cmd.Run()
	//err := cmd.Start()
	if err != nil {
		fmt.Errorf("err %v \n",err)
	}
	fmt.Printf("Waiting for command to finish... \n")
	// Start，上面的内容会先输出，然后这里会阻塞等待5秒
	err = cmd.Wait()
	fmt.Printf("Command finished with error: %v", err)
}
```
