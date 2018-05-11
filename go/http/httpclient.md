## 自定义http 请求
先介绍一下RoundTrip的使用，它是一个接口，是发送http的载体，我们自己可以去加上自己定制的请求
下面例子，是我们可以通用截获所以的http请求，并加上basic认证的代码
```go
package main
import (
"encoding/base64"
"fmt"
"net/http"
	"io/ioutil"
)

type BasicAuthTransport struct {
   Username string
   Password string
}

func (bat BasicAuthTransport) RoundTrip(req *http.Request) (*http.Response, error) {
   req.Header.Set("Authorization", fmt.Sprintf("Basic %s",
   base64.StdEncoding.EncodeToString([]byte(fmt.Sprintf("%s:%s",
   bat.Username, bat.Password)))))
   return http.DefaultTransport.RoundTrip(req)
}

func (bat *BasicAuthTransport) Client() *http.Client {
	return &http.Client{Transport: bat}
}

func main() {
	tansport := &BasicAuthTransport{
		"bac",
		"123",
	}
	client := tansport.Client()
	req, _ := http.NewRequest("GET","https://xxx.xxx.cn", nil)
	resp ,_ := client.Do(req)
	fmt.Println(req.Header)
	b, _ := ioutil.ReadAll(resp.Body)
	fmt.Println(string(b))
}
```
通过上面的方式可以实现添加添加请求头的功能，
如你的请求头比较多，你也可以分别添加，如下
先定义一个修改的方法，这个方法是加上请求的客户端
```go
type userAgentModifier struct {
	userAgent string
}

// Modify adds user-agent header to the request
func (u *userAgentModifier) Modify(req *http.Request) error {
	req.Header.Set(http.CanonicalHeaderKey("User-Agent"), u.userAgent)
	return nil
}
```
再定义一个修改的方法，这个方法是加上认证信息的
```go
type AuthorizerStore struct {
	authorizers []Authorizer
	ping        *url.URL
	challenges  []challenge.Challenge
}

func (a *AuthorizerStore) Modify(req *http.Request) error {
	//only handle the requests sent to registry
	v2Index := strings.Index(req.URL.Path, "/v2/")
	if v2Index == -1 {
		return nil
	}
	fmt.Printf("call AuthorizerStore Modify \n")
	ping := url.URL{
		Host:   req.URL.Host,
		Scheme: req.URL.Scheme,
		Path:   req.URL.Path[:v2Index+4],
	}

	if ping.Host != a.ping.Host || ping.Scheme != a.ping.Scheme ||
		ping.Path != a.ping.Path {
		return nil
	}

	for _, challenge := range a.challenges {
		for _, authorizer := range a.authorizers {
			if authorizer.Scheme() == challenge.Scheme {
				if err := authorizer.Authorize(req, challenge.Parameters); err != nil {
					return err
				}
			}
		}
	}

	return nil
}
```
这样我们在定义上面例子里面的方法
```go
func (t *Transport) RoundTrip(req *http.Request) (*http.Response, error) {
	fmt.Println("call  RoundTrip")
	for _, modifier := range t.modifiers {
		fmt.Printf("modifier :%v \n",modifier)
		if err := modifier.Modify(req); err != nil {
			return nil, err
		}
	}

	resp, err := t.transport.RoundTrip(req)
	if err != nil {
		return nil, err
	}

	log.Debugf("%d | %s %s", resp.StatusCode, req.Method, req.URL.String())

	return resp, err
}

```
截获请求，调用不同修改方法，自定义请求！





## GET

```go
func httpGet() {
    resp, err := http.Get("http://www.01happy.com/demo/accept.php?id=1")
    if err != nil {
        // handle error
    }
 
    defer resp.Body.Close()
    body, err := ioutil.ReadAll(resp.Body)
    if err != nil {
        // handle error
    }
 
    fmt.Println(string(body))
}
```

## POST
```go
data, err := json.Marshal(struct)
//或者
str := `{"a":"b"}`
date = []byte(str)
req, err := http.NewRequest("POST", url, bytes.NewReader(data))


func httpPost() {
    resp, err := http.Post("http://www.01happy.com/demo/accept.php",
        "application/x-www-form-urlencoded",
        strings.NewReader("name=cjb"))
    if err != nil {
        fmt.Println(err)
    }
 
    defer resp.Body.Close()
    body, err := ioutil.ReadAll(resp.Body)
    if err != nil {
        // handle error
    }
 
    fmt.Println(string(body))
}

```
设置请求头
```go
req.Header.Set("Authorization", "token ltguvneekiynjbdsijmzmyarjovgycygxjqcfaigkejpobpj")
```

例如：
```go
func SendGet(url string)([]byte, error){
	resp, err := http.Get(url)
	if err != nil{
		return nil, err
	}
	defer resp.Body.Close()
	body, err := ioutil.ReadAll(resp.Body)
	if err != nil{
		return nil, err
	}
	return body, nil
}


func SendPost(url string,req []byte)([]byte, error){
	body_type := "application/json;charset=utf-8"
	resp, err := http.Post(url, body_type, bytes.NewBuffer(req))
	if err != nil{
		return nil, err
	}
	defer resp.Body.Close()
	body, err := ioutil.ReadAll(resp.Body)
	if err != nil{
		return nil, err
	}
	return body, nil
}
```