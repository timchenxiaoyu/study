## filter 查询

### filter查询

```go
docker ps --filter "label=io.kubernetes.docker.type=container"|awk '{print $1}'


docker ps --filter volume=/var/lib/kubelet/pods/e4b792a9-fa0b-11e7-9d36-5254b24cbf5e/etc-hosts

```

### 获取volume的容器
在volume（/var/lib/docker/volumes）下执行
```go
ls |awk '{print  "/var/lib/docker/volumes/"$1}'|awk '{print  system("docker ps --filter volume="$1)}'
```