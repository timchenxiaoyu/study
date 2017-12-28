## 第一个go http server

下面是一个go hello world 的demo
```go
package main

import (
	"io"
	"net/http"
)

func main() {

	http.HandleFunc("/", sayhello)
	http.HandleFunc("/say", sayhello1)
	http.ListenAndServe(":8080", nil)
}

func sayhello(w http.ResponseWriter, r *http.Request) {
	io.WriteString(w, "hello world")
}

func sayhello1(w http.ResponseWriter, r *http.Request) {
	io.WriteString(w, "hello world1")
}

type middleHandle struct {
	handle http.Handler
}
```
启动并测试
```
curl 127.0.0.1:8080
hello world
------------
curl 127.0.0.1:8080/say
hello world1
```