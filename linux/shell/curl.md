## curl

### POST
默认是application/x-www-form-urlencoded
对于表单提交
```go
curl -d "param1=value1&param2=value2" -X POST http://localhost:3000/data
```
等效于
```go
curl -d "param1=value1&param2=value2" -H "Content-Type: application/x-www-form-urlencoded" -X POST http://localhost:3000/data
```
如果是jsont提交则是
```go
curl -d '{"key1":"value1", "key2":"value2"}' -H "Content-Type: application/json" -X POST http://localhost:3000/data
```
如果是json文件也是可以的
```go
curl -d "@data.json" -X POST http://localhost:3000/data

data.json内容如下
{
  "key1":"value1",
  "key2":"value2"
}

```


### 上传文件
```go
curl -F "uploadfile=@C:\Users\wangs\Desktop\xueba_license.txt" localhost:8080/upload
```