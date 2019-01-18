# nc

## 安装
```
yum install nmap-ncat.x86_64
```

## 使用

### HTTP
启动HTTP 服务端
```
 while true ; do  echo -e "HTTP/1.1 200 OK\n\n $(date)" | nc -l -p 1500  ; done
```
发送HTTP请求
```
nc 10.100.139.241 1500 <<EOF

GET / HTTP/1.0

EOF
```

### TCP
服务端
```
cat file|nc -l -p 8888
```
客户端
```
nc 192.168.1.11 8888 >file
```