## flag
获取用户的输入
```go

package main

import "flag"
import "fmt"

var ip string
var port int

func init() {
	flag.StringVar(&ip, "ip", "0.0.0.0", "ip address")
	flag.IntVar(&port, "port", 8000, "port number")
}

func main() {
	flag.Parse()
	fmt.Printf("%s:%d", ip, port)
}
```
其中flag.Parse()是执行转化的关键步骤，不可省略
编译并运行通过  -ip=  -port= 输入参数