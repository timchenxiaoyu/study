# memory

限制cgroup中所有进程所能使用的物理内存总量
限制cgroup中所有进程所能使用的物理内存+交换空间总量
限制cgroup中所有进程所能使用的内核内存总量及其它一些内核资源

```go
 cgroup.event_control       #用于eventfd的接口
 memory.usage_in_bytes      #显示当前已用的内存
 memory.limit_in_bytes      #设置/显示当前限制的内存额度
 memory.failcnt             #显示内存使用量达到限制值的次数
 memory.max_usage_in_bytes  #历史内存最大使用量
 memory.soft_limit_in_bytes #设置/显示当前限制的内存软额度
 memory.stat                #显示当前cgroup的内存使用情况
 memory.use_hierarchy       #设置/显示是否将子cgroup的内存使用情况统计到当前cgroup里面
 memory.force_empty         #触发系统立即尽可能的回收当前cgroup中可以回收的内存
 memory.pressure_level      #设置内存压力的通知事件，配合cgroup.event_control一起使用
 memory.swappiness          #设置和显示当前的swappiness
 memory.move_charge_at_immigrate #设置当进程移动到其他cgroup中时，它所占用的内存是否也随着移动过去
 memory.oom_control         #设置/显示oom controls相关的配置
 memory.numa_stat           #显示numa相关的内存
```

测试
```go
# cd /sys/fs/cgroup/memory/test/
# sh -c "echo $$ >> cgroup.procs"
# top

打开一个新的窗口
# cat tasks 
1289
1482

# cat  memory.limit_in_bytes    
9223372036854771712

# cat  memory.usage_in_bytes
831488

```
可以看到使用了831K的内存，现在开始限制它的内存
```go
# sudo sh -c "echo 400K > memory.limit_in_bytes"  如果是-1表示不限制
# free
              total        used        free      shared  buff/cache   available
Mem:        4046420       63532     3836216        8428      146672     3775948
Swap:       4194300           0     4194300
# top

在第二个窗口
# cat memory.limit_in_bytes 
409600
# cat memory.usage_in_bytes 
389120

# free
              total        used        free      shared  buff/cache   available
Mem:        4046420       63788     3835560        8428      147072     3775468
Swap:       4194300         560     4193740
```
可以看到内存被限制，只能使用交换分区并且频繁的达到内存限制
```go
# cat memory.failcnt
11211

#  cat memory.stat
total_pgpgin 43115
total_pgpgout 43016
total_pgfault 35004
```
频繁的换进换出

## memory.oom_control

这个文件里面包含了一个控制是否为当前cgroup启动OOM-killer的标识。如果写0到这个文件，将启动OOM-killer，当内核无法给进程分配足够的内存时，
将会直接kill掉该进程；如果写1到这个文件，表示不启动OOM-killer，当内核无法给进程分配足够的内存时，
将会暂停该进程直到有空余的内存之后再继续运行；同时，memory.oom_control还包含一个只读的under_oom字段，
用来表示当前是否已经进入oom状态，也即是否有进程被暂停了。

写一个测试程序
```go
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>

#define MB (1024 * 1024)

int main(int argc, char *argv[])
{
    char *p;
    int i = 0;
    while(1) {
        p = (char *)malloc(MB);
        memset(p, 0, MB);
        printf("%dM memory allocated\n", ++i);
        sleep(1);
    }

    return 0;
}
```
编译测试
```go
# sudo sh -c "echo 5M > memory.limit_in_bytes"
#  cat memory.oom_control
oom_kill_disable 0
under_oom 0
# sudo sh -c "echo 0 > memory.swappiness"

第二个窗口
# sudo sh -c "echo $$ >> cgroup.procs"
# gcc ~/mem-allocate.c -o ~/mem-allocate
#  ~/mem-allocate
1M memory allocated
2M memory allocated
3M memory allocated
4M memory allocated
Killed


```
上面的测试必须要关闭交换分区，否则会一直从交换分区申请内存

如果设置
```go
# sudo sh -c "echo 1 >> memory.oom_control"
```
那么程序会一直hang住
```go
#  ~/mem-allocate
1M memory allocated
2M memory allocated
3M memory allocated
4M memory allocated

```
不会被kill