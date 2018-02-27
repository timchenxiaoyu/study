# swap

swap space是磁盘上的一块区域，可以是一个分区，也可以是一个文件
。简单点说，当系统物理内存吃紧时，Linux会将内存中不常访问的数据保存到swap上，这样系统就有更多的物理内存为各个进程服务，
而当系统需要访问swap上存储的内容时，再将swap上的数据加载到内存中，这就是我们常说的swap out和swap in。
```go
swapon -s
文件名				类型		大小	已用	权限
/dev/vdb                               	partition	2097148	1667064	-1
```


## 关闭swap
```go
# sudo swapoff -a
# swapon -s
# free
```

## 配置swappiness

有时我们桌面环境确实配置了比较充裕的内存，并且也配置了swap空间，这个时候就希望尽量减少swap空间的使用，避免对系统性能造成影响，
Linux早就帮我们考虑到这种情况了，在2.6内核中，增加了一个叫做swappiness的参数，用于配置需要将内存中不常用的数据移到swap中去的紧迫程度。
这个参数的取值范围是0～100，0告诉内核尽可能的不要将内存数据移到swap中，也即只有在迫不得已的情况下才这么做，而100告诉内核只要有可能，
尽量的将内存中不常访问的数据移到swap中。

Ubuntu的desktop和server的默认配置都是60(可能会随着版本变化)，对于桌面环境来说，界面的响应速度直接关系到系统的流畅程度，
如果内存比较充裕的话，可以将这个值设置的小一点，这样就尽可能的把数据留在内存中，从而唤醒后台界面程序会更快一些，
Ubuntu desktop建议将该值设置为10，当然大家可以根据swap空间的实际使用情况，任意调整这个参数，直到自己满意的水平为止。
对于服务器来说，主要性能衡量标准是整体的处理能力，而不是具体某一次的响应速度，能把更多的内存用来做I/O cache可能效果更好，
所以Ubuntu server建议保持60的默认值。
```go
#  cat /proc/sys/vm/swappiness
60

```

临时修改当前系统中swappiness的值
```go
#  sudo sysctl vm.swappiness=50
vm.swappiness = 50
#  cat /proc/sys/vm/swappiness
50

```
永久修改
```go
/etc/sysctl.conf下
vm.swappiness=50
```