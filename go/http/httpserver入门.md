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
上面是通过HandleFunc，未免有点局限，如果你在多个地方实现ServeHTTP这个接口都可以注册到http服务端，如下
```go
package main

import (
    "fmt"
    "net/http"
)

type boy struct{}

func (b boy) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintln(w, "I love you!!!")
}

type girl struct{}

func (g girl) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintln(w, "hehe.")
}

type foo struct{}

func (f foo) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintln(w, "Men would stop talking and women would shed tears when they see this.")
}

func main() {
    var b boy
    var g girl
    var f foo

    http.Handle("/boy/", b)
    http.Handle("/girl/", g)
    http.Handle("/", f)
    http.ListenAndServe(":8080", nil)
}
```
运行并测试
```go

$ curl 127.0.0.1:8080/boy
I love you!!!

$ curl 127.0.0.1:8080/girl
hehe.
$ curl 127.0.0.1:8080
Men would stop talking and women would shed tears when they see this.
```