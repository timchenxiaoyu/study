在入门里面我们直接通过http.ListenAndServe(":8080", nil)的方式直接启动，但这样实现毕竟太简单，如果想做更负责的分发
就可以自己去实现
如果深入看一下这个方法就可以发现
```go
func ListenAndServe(addr string, handler Handler) error {
	server := &Server{Addr: addr, Handler: handler}
	return server.ListenAndServe()
}
```
这个方法接受传参handler，那么handler是什么呢？
```go
type Handler interface {
	ServeHTTP(ResponseWriter, *Request)
}
```
就是一个实现了http请求并返回的接口，那么是不是任何实现了这个接口的都可以呢？答案是肯定的
下面举个例子
```go
package main

import (
	"fmt"
	"net/http"
)
// 创建一个 foo 类型
type foo struct {}
// 为 foo 类型创建 ServeHTTP 方法，以实现 Handle 接口
func (f foo) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintln(w, "Implement the Handle interface.")
}

func main() {
	// 创建对象，类型名写后面..
	var f foo
	http.ListenAndServe(":8080",f)
}
```
启动并测试
```go
$ curl 127.0.0.1:8080
Implement the Handle interface.

```
那么如果你想实现更加复杂的路由，都可以在ServeHTTP这个方法里面去实现了，譬如下面这个例子
```go
package main

import (
	"fmt"
	"net/http"
)
// 创建一个 foo 类型
type foo struct {}
// 为 foo 类型创建 ServeHTTP 方法，以实现 Handle 接口
func (f foo) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	// 根据 URL 的相对路径来设置网页内容（不优雅）
	switch r.URL.Path {
	case "/boy":
		fmt.Fprintln(w, "I love you!!!")
	case "/girl":
		fmt.Fprintln(w, "hehe.")
	default:
		fmt.Fprintln(w, "Men would stop talking and women would shed tears when they see this.")
	}
}
func main() {
	// 创建对象，类型名写后面..
	var f foo
	http.ListenAndServe(":8080",f)
}
```
这通通过url path去判断该使用哪条路由，测试一下：
```go

$ curl 127.0.0.1:8080/boy
I love you!!!

$ curl 127.0.0.1:8080/girl
hehe.
$ curl 127.0.0.1:8080
Men would stop talking and women would shed tears when they see this.

```
如果你认为实现不够优雅可以通过http.NewServeMux() 优化，
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

	// 返回一个 *ServeMux 对象
	mux := http.NewServeMux()
	mux.Handle("/boy/", b)
	mux.Handle("/girl/", g)
	mux.Handle("/", f)
	http.ListenAndServe(":8080", mux)
}
```

## 深入理解

### HandleFunc

HandleFunc需要传入的是一个函数 
```go
handler func(ResponseWriter, *Request)
```
并把它放到DefaultServeMux里面，二所谓的HandlerFunc就是这个方法的别名
```go
type HandlerFunc func(ResponseWriter, *Request)
```

所以如果想实现级联就只需要对原理的方法进行包装就可以了，譬如
```go
http.HandleFunc("/say",NeedLogin(say))
...
func NeedLogin(h http.HandlerFunc) http.HandlerFunc{
	return func(w http.ResponseWriter, r *http.Request) {
		fmt.Println("get request")
		h(w,r)
	}
}
```
这样就可以在线执行NeedLogin方法了，上面的h就是say。执行完NeedLogin再去执行h方法。

### Handle
```go
http.ListenAndServe(":9999",r)
```
这个方法需要传入的是Handler,而Handler又是什么呢？它是一个接口，实现了ServeHTTP的一个接口
```go
type Handler interface {
	ServeHTTP(ResponseWriter, *Request)
}
```
那么只要是实现了它的接口都可以放入，上文已经举例说明了

那么如果你想实现级联，同样可以

```go
type Counter struct {
	count map[string]int
	h   http.Handler
}

func NewCounter(h http.Handler) *Counter{
	return &Counter{
		h :h,
		count: make(map[string]int ),
	}
}

func (c *Counter)ServeHTTP(w http.ResponseWriter, r *http.Request){
	c.count[r.URL.Path]++
	c.h.ServeHTTP(w,r)
}
//----------------------------------------------//

type Recorder struct {
	h   http.Handler
}

func NewRecoder(h http.Handler) *Recorder{
	return &Recorder{
		h :h,

	}
}

func (c *Recorder)ServeHTTP(w http.ResponseWriter, r *http.Request){
	fmt.Println("record ", r.URL.Path)
	c.h.ServeHTTP(w,r)
}


//------------------------------------------------//
//h := handlers.LoggingHandler(os.Stderr,http.DefaultServeMux)
	c := NewCounter(http.DefaultServeMux)//this switch http.DefaultServeMux to h is ok,add log feature
	r := NewRecoder(c)

	http.ListenAndServe(":9999",r)
```

上面通过层层包装先是NewCounter、然后NewRecoder，这样在调用9999端口的这个服务的时候都会先执行上面两个方法

