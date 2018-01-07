## pivot_root

pivot_root 修改当root文件系统路径

```go
pivot_root new_root put_old  
```
new_root 是新的root fs路径
put_old  将当前进程的rootfs保存到这个目录下面

for example

```go
docker run -it -v /root/kkkk:/test --privileged=true busybox sh

cd /test

mkdir oldroot/

pivot_root . oldroot/

```

