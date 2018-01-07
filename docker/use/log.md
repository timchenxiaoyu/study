
## 日志导出

### JSON
docker的log可以设定指定的输出方式，通常为了日志采集我们使用json文件方法是输出，这个是docker daemon的默认日志输出方式
首先设置/etc/docker/daemon.json 
```go
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m"
  }
}
```
参数解释：
max-size：文件大小设定，设定上限
max-file：最大文件数


也可以单独覆盖设置
```go
docker run -it --log-opt max-size=10m --log-opt max-file=3 alpine ash

```

### Syslog
也可以通过syslog的方式统一采集
先启动一个syslog
```go
docker run -d -v /tmp:/var/log/syslog -p 127.0.0.1:5514:514/udp  --name rsyslog voxxit/rsyslog
```
然后启动业务容器
```go
docker run --log-driver=syslog --log-opt syslog-address=udp://127.0.0.1:5514 --log-opt syslog-facility=daemon 
 --log-opt syslog-tag=app01 --name logging-02 -ti -d -v $(pwd):/tmp  -w /tmp python:2.7 python -u logging-01.py
```
syslog-tag:设定来自这个container的日志的标识
syslog-address：设定syslog的地址
此时业务容器直接输出通过docker logs的方式是不能输出日志的"logs" command is supported only for "json-file" logging driver 
测试一下：
```go
docker exec  rsyslog tail -f /var/log/messages
```

注：python脚本如下
```go
import sys  
import time  
 while True:  
   sys.stderr.write('Error\n')  
   sys.stdout.write('All Good\n')  
   time.sleep(1)
```