## url 操作
将普通的字符串转化为url操作
```go
u,_ := url.Parse("https://pic3.zhimg.com/v2-28fd28c8a5ffc58b6ff5e9b1d2c586fe.jpg?a=1")
fmt.Println(u.Scheme,"****",u.Host,"*****",u.Path,"********",u.RawQuery)
```
输出
```go
https **** pic3.zhimg.com ***** /v2-28fd28c8a5ffc58b6ff5e9b1d2c586fe.jpg ******** a=1
```