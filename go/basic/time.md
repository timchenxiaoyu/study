## time

时间加减
```
time.Now().Add(-1*time.Minute)
```

## 格式转化

时间转字符串
```go
now := time.Now()
formatNow := now.Format("2006-01-02 15:04:05")
fmt.Println(formatNow)
```

字符串转时间
```go
local, _ := time.LoadLocation("Local")
t, _ := time.ParseInLocation("2006-01-02 15:04:05", "2017-06-20 18:16:15", local)
fmt.Println(t)

```
这个里面需要注意，上面的2006-01-02 15:04:05不是随意的，而是确定写死的，这个需要注意
第二如果不传Local（服务器本地时区）就是UTC时间