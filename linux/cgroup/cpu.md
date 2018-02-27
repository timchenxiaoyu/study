# CPU

其中cpuset主要用于设置CPU的亲和性，可以限制cgroup中的进程只能在指定的CPU上运行，或者不能在指定的CPU上运行，同时cpuset还能设置内存的亲和性。

```go
# mount|grep cpu
cgroup on /sys/fs/cgroup/cpu,cpuacct type cgroup (rw,nosuid,nodev,noexec,relatime,cpu,cpuacct)
cgroup on /sys/fs/cgroup/cpuset type cgroup (rw,nosuid,nodev,noexec,relatime,cpuset)
```

## cpu.cfs_period_us & cpu.cfs_quota_us

cfs_period_us用来配置时间周期长度，cfs_quota_us用来配置当前cgroup在设置的周期长度内所能使用的CPU时间数，两个文件配合起来设置CPU的使用上限。
两个文件的单位都是微秒（us），cfs_period_us的取值范围为1毫秒（ms）到1秒（s），cfs_quota_us的取值大于1ms即可，
如果cfs_quota_us的值为-1（默认值），表示不受cpu时间的限制。下面是几个例子：
```go
1.限制只能使用1个CPU（每250ms能使用250ms的CPU时间）
    # echo 250000 > cpu.cfs_quota_us /* quota = 250ms */
    # echo 250000 > cpu.cfs_period_us /* period = 250ms */

2.限制使用2个CPU（内核）（每500ms能使用1000ms的CPU时间，即使用两个内核）
    # echo 1000000 > cpu.cfs_quota_us /* quota = 1000ms */
    # echo 500000 > cpu.cfs_period_us /* period = 500ms */

3.限制使用1个CPU的20%（每50ms能使用10ms的CPU时间，即使用一个CPU核心的20%）
    # echo 10000 > cpu.cfs_quota_us /* quota = 10ms */
    # echo 50000 > cpu.cfs_period_us /* period = 50ms */
```

## cpu.shares
shares用来设置CPU的相对值，并且是针对所有的CPU（内核），默认值是1024，假如系统中有两个cgroup，分别是A和B，A的shares值是1024，B的shares值是512，那么A将获得1024/(1204+512)=66%的CPU资源，而B将获得33%的CPU资源。shares有两个特点：

如果A不忙，没有使用到66%的CPU时间，那么剩余的CPU时间将会被系统分配给B，即B的CPU使用率可以超过33%

如果添加了一个新的cgroup C，且它的shares值是1024，那么A的限额变成了1024/(1204+512+1024)=40%，B的变成了20%

从上面两个特点可以看出：

在闲的时候，shares基本上不起作用，只有在CPU忙的时候起作用，这是一个优点。

由于shares是一个绝对值，需要和其它cgroup的值进行比较才能得到自己的相对限额，而在一个部署很多容器的机器上，cgroup的数量是变化的，所以这个限额也是变化的，自己设置了一个高的值，但别人可能设置了一个更高的值，所以这个功能没法精确的控制CPU使用率。

## cpu.stat

包含了下面三项统计结果

nr_periods： 表示过去了多少个cpu.cfs_period_us里面配置的时间周期

nr_throttled： 在上面的这些周期中，有多少次是受到了限制（即cgroup中的进程在指定的时间周期中用光了它的配额）

throttled_time: cgroup中的进程被限制使用CPU持续了多长时间(纳秒)

测试
```go
# cd /sys/fs/cgroup/cpu,cpuacct
# mkdir test
# sudo sh -c "echo 50000 > cpu.cfs_period_us"
# sudo sh -c "echo 10000 > cpu.cfs_quota_us"
# sudo sh -c "echo $$ > cgroup.procs"
# while :; do echo test > /dev/null; done

打开新的窗口
# top
%Cpu(s):  8.4 us,  2.6 sy,  0.0 ni, 88.9 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND                                                                                                                                
 4125 root      20   0   19988   3748   3180 R  19.9  0.1   0:22.39 bash  

```
cpu被控制在单核的20%, 整个系统占用8.4+2.6 超过10%多一点，因为我的系统是2核的，所有是10%


* 总结
使用cgroup限制CPU的使用率比较纠结，用cfs_period_us & cfs_quota_us吧，限制死了，没法充分利用空闲的CPU，用shares吧，又没法配置百分比，
极其难控制。总之，使用cgroup的cpu子系统需谨慎。