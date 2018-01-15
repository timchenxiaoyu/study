## Docker registry 完全解析

### 拉取镜像
* 1.获取镜像manifests
```go
GET /v2/<name>/manifests/<reference>
```
name是镜像名称，reference是镜像版本或者digest

* 2.拉取层

```go
GET /v2/<name>/blobs/<digest>
```
name是镜像名称，digest是通过manifest获取的数字串

### 上传镜像
* 1.申请上传
```go
POST /v2/<name>/blobs/uploads/
```
* 2.校验blob是否存在
```go
HEAD /v2/<name>/blobs/<digest>
```
* 3.上传层
单片上传
```go
PUT /v2/<name>/blobs/uploads/<uuid>?digest=<digest>
Content-Length: <size of layer>
Content-Type: application/octet-stream
```
分段上传
```go
PATCH /v2/<name>/blobs/uploads/<uuid>
Content-Length: <size of chunk>
Content-Range: <start of range>-<end of range>
Content-Type: application/octet-stream
```
* 4.上传完毕去人

```go
PUT /v2/<name>/blob/uploads/<uuid>?digest=<digest>
```

## 删除镜像
* 1.删除manifest
```go
DELETE /v2/<name>/manifests/<reference>
```
