如果有socket编程经验的人，对于启动一个tcp的本地监听应该再熟悉不过，不赘述了看下面一个helloworld的例子
```go
package main

import (
    "fmt"
    "log"
    "net"
)

func main() {
    // net 包提供方便的工具用于 network I/O 开发，包括TCP/IP, UDP 协议等。
    // Listen 函数会监听来自 8080 端口的连接，这里也可以填写地址：端口这样只会监听访问该地址+端口的请求
    // 返回一个 net.Listener 对象。
    li, err := net.Listen("tcp", ":8080")
    // 错误处理
    if err != nil {
        log.Panic(err)
    }
    // 释放连接，通过 defer 关键字可以让连接在函数结束前进行释放
    // 这样可以不关心释放资源的语句位置，增加代码可读性
    defer li.Close()

    // 不断循环，不断接收来自客户端的请求
    for {
        // Accept 函数会阻塞程序，直到接收到来自端口的连接
        // 每接收到一个链接，就会返回一个 net.Conn 对象表示这个连接
        conn, err := li.Accept()

        if err != nil {
            log.Println(err)
        }
        // 字符串写入到客户端
        fmt.Fprintln(conn, "Hello from TCP server")

        conn.Close()
    }
}
```
运行并测试
```go
$ telnet 127.0.0.1 8080
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
Hello from TCP server
Connection closed by foreign host.

```

也可以自己写一个客户端测试
```go
package main

import (
	"fmt"
	"io/ioutil"
	"log"
	"net"
)

func main() {
	// net 包的 Dial 函数能创建一个 TCP 连接
	conn, err := net.Dial("tcp", ":8080")
	if err != nil {
		log.Fatal(err)
	}
	// 别忘了关闭连接
	defer conn.Close()
	// 通过 ioutil 来读取连接中的内容，返回一个 []byte 类型的对象
	byte, err := ioutil.ReadAll(conn)
	if err != nil {
		log.Println(err)
	}
	// []byte 类型的数据转成字符串型，再将其打印输出
	fmt.Println(string(byte))
}
```


## 文件服务器
如果要实现文件服务器的上传和下载需要通过
```go
io.copy(conn,file)
```
和

```go
r := bufio.newRead(conn)
io.copy(file,r)
```
尤其是上传部分部分一定要使用缓冲流