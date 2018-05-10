## path

```go
fmt.Println(path.Base("https://pic3.zhimg.com/v2-28fd28c8a5ffc58b6ff5e9b1d2c586fe.jpg"))

```
返回
```go
v2-28fd28c8a5ffc58b6ff5e9b1d2c586fe.jpg
```
也就是最后一个/后面的部分
其实现的原理很简单
```go
if i := strings.LastIndex(path, "/"); i >= 0 {
		path = path[i+1:]
	}
```
找到最后一个"/"，然后通过字符串的切割完成