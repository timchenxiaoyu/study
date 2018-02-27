cgroup和namespace类似，也是将进程进行分组，但它的目的和namespace不一样，namespace是为了隔离进程组之间的资源，
而cgroup是为了对一组进程进行统一的资源监控和限制。

资源限制（Resource Limitation）：cgroups可以对进程组使用的资源总额进行限制。如设定应用运行时使用内存的上限，一旦超过这个配额就发出OOM（Out of Memory）。
优先级分配（Prioritization）：通过分配的CPU时间片数量及硬盘IO带宽大小，实际上就相当于控制了进程运行的优先级。
资源统计（Accounting）： cgroups可以统计系统的资源使用量，如CPU使用时长、内存用量等等，这个功能非常适用于计费。
进程控制（Control）：cgroups可以对进程组执行挂起、恢复等操作。


* task（任务）：cgroups的术语中，task就表示系统的一个进程。
* cgroup（控制组）：cgroups 中的资源控制都以cgroup为单位实现。cgroup表示按某种资源控制标准划分而成的任务组，包含一个或多个子系统。
一个任务可以加入某个cgroup，也可以从某个cgroup迁移到另外一个cgroup。
* subsystem（子系统）：cgroups中的subsystem就是一个资源调度控制器（Resource Controller）。比如CPU子系统可以控制CPU时间分配，内存子系统可以限制cgroup内存使用量。
* hierarchy（层级树）：hierarchy由一系列cgroup以一个树状结构排列而成，每个hierarchy通过绑定对应的subsystem进行资源调度。
hierarchy中的cgroup节点可以包含零或多个子节点，子节点继承父节点的属性。整个系统可以有多个hierarchy。



同一个hierarchy可以附加一个或多个subsystem。如下图1，cpu和memory的subsystem附加到了一个hierarchy。

![同一个hierarchy可以附加一个或多个subsystem](../../images/pic1.png)

当前Linux自动挂载的cgroup
```go
# mount|grep cgroup
tmpfs on /sys/fs/cgroup type tmpfs (ro,nosuid,nodev,noexec,mode=755)
cgroup on /sys/fs/cgroup/systemd type cgroup (rw,nosuid,nodev,noexec,relatime,xattr,release_agent=/lib/systemd/systemd-cgroups-agent,name=systemd)
cgroup on /sys/fs/cgroup/net_cls,net_prio type cgroup (rw,nosuid,nodev,noexec,relatime,net_cls,net_prio)
cgroup on /sys/fs/cgroup/cpuset type cgroup (rw,nosuid,nodev,noexec,relatime,cpuset)
cgroup on /sys/fs/cgroup/perf_event type cgroup (rw,nosuid,nodev,noexec,relatime,perf_event)
cgroup on /sys/fs/cgroup/blkio type cgroup (rw,nosuid,nodev,noexec,relatime,blkio)
cgroup on /sys/fs/cgroup/memory type cgroup (rw,nosuid,nodev,noexec,relatime,memory)
cgroup on /sys/fs/cgroup/hugetlb type cgroup (rw,nosuid,nodev,noexec,relatime,hugetlb)
cgroup on /sys/fs/cgroup/pids type cgroup (rw,nosuid,nodev,noexec,relatime,pids)
cgroup on /sys/fs/cgroup/freezer type cgroup (rw,nosuid,nodev,noexec,relatime,freezer)
cgroup on /sys/fs/cgroup/cpu,cpuacct type cgroup (rw,nosuid,nodev,noexec,relatime,cpu,cpuacct)
cgroup on /sys/fs/cgroup/devices type cgroup (rw,nosuid,nodev,noexec,relatime,devices)
```
* cpu (since Linux 2.6.24; CONFIG_CGROUP_SCHED)
用来限制cgroup的CPU使用率。

* cpuacct (since Linux 2.6.24; CONFIG_CGROUP_CPUACCT)
统计cgroup的CPU的使用率。

* cpuset (since Linux 2.6.24; CONFIG_CPUSETS)
绑定cgroup到指定CPUs和NUMA节点。

* memory (since Linux 2.6.25; CONFIG_MEMCG)
统计和限制cgroup的内存的使用率，包括process memory, kernel memory, 和swap。

* devices (since Linux 2.6.26; CONFIG_CGROUP_DEVICE)
限制cgroup创建(mknod)和访问设备的权限。

* freezer (since Linux 2.6.28; CONFIG_CGROUP_FREEZER)
suspend和restore一个cgroup中的所有进程。

* net_cls (since Linux 2.6.29; CONFIG_CGROUP_NET_CLASSID)
将一个cgroup中进程创建的所有网络包加上一个classid标记，用于tc和iptables。 只对发出去的网络包生效，对收到的网络包不起作用。

* blkio (since Linux 2.6.33; CONFIG_BLK_CGROUP)
限制cgroup访问块设备的IO速度。

* perf_event (since Linux 2.6.39; CONFIG_CGROUP_PERF)
对cgroup进行性能监控

* net_prio (since Linux 3.3; CONFIG_CGROUP_NET_PRIO)
针对每个网络接口设置cgroup的访问优先级。

* hugetlb (since Linux 3.5; CONFIG_CGROUP_HUGETLB)
限制cgroup的huge pages的使用量。

* pids (since Linux 4.3; CONFIG_CGROUP_PIDS)
限制一个cgroup及其子孙cgroup中的总进程数。

通过命令行查看
```
apt-get install cgroup-bin

查看所以的cgroup
lscgroup
cpuset:/
cpu:/
cpuacct:/
memory:/
devices:/
freezer:/
blkio:/
perf_event:/
hugetlb:/

```
查看所以支持的子系统
```
lssubsys -a
cpuset
cpu
cpuacct
memory
devices
freezer
blkio
perf_event
hugetlb

```

查看所有子系统挂载的位置  lssubsys –m

查看单个子系统（如memory）挂载位置：lssubsys –m memory


* 第一次挂载一颗和指定subsystem关联的cgroup树时，会创建一颗新的cgroup树，当再一次用同样的参数挂载时，
会重用现有的cgroup树，也即两个挂载点看到的内容是一样的。

```go
# sudo mkdir /sys/fs/cgroup/cpu,cpuacct/test
# ls -l /sys/fs/cgroup/cpu,cpuacct/test
total 0
-rw-r--r-- 1 root root 0 Feb  8 11:18 cgroup.clone_children
-rw-r--r-- 1 root root 0 Feb  8 11:18 cgroup.procs
-r--r--r-- 1 root root 0 Feb  8 11:18 cpuacct.stat
-rw-r--r-- 1 root root 0 Feb  8 11:18 cpuacct.usage
-r--r--r-- 1 root root 0 Feb  8 11:18 cpuacct.usage_percpu
-rw-r--r-- 1 root root 0 Feb  8 11:18 cpu.cfs_period_us
-rw-r--r-- 1 root root 0 Feb  8 11:18 cpu.cfs_quota_us
-rw-r--r-- 1 root root 0 Feb  8 11:18 cpu.shares
-r--r--r-- 1 root root 0 Feb  8 11:18 cpu.stat
-rw-r--r-- 1 root root 0 Feb  8 11:18 notify_on_release
-rw-r--r-- 1 root root 0 Feb  8 11:18 tasks

# mkdir -p ./cgroup/cpu,cpuacct && cd ./cgroup/
# mount -t cgroup -o cpu,cpuacct new-cpu-cpuacct ./cpu,cpuacct
# ll 
dr-xr-xr-x  6 root root    0 Feb  8 11:18 ./
drwxr-xr-x  3 root root 4096 Feb  8 11:21 ../
-rw-r--r--  1 root root    0 Feb  8 11:18 cgroup.clone_children
-rw-r--r--  1 root root    0 Feb  8 11:18 cgroup.procs
-r--r--r--  1 root root    0 Feb  8 11:18 cgroup.sane_behavior
-r--r--r--  1 root root    0 Feb  8 11:18 cpuacct.stat
-rw-r--r--  1 root root    0 Feb  8 11:18 cpuacct.usage
-r--r--r--  1 root root    0 Feb  8 11:18 cpuacct.usage_percpu
-rw-r--r--  1 root root    0 Feb  8 11:18 cpu.cfs_period_us
-rw-r--r--  1 root root    0 Feb  8 11:18 cpu.cfs_quota_us
-rw-r--r--  1 root root    0 Feb  8 11:18 cpu.shares
-r--r--r--  1 root root    0 Feb  8 11:18 cpu.stat
drwxr-xr-x  2 root root    0 Feb  8 11:18 init.scope/
-rw-r--r--  1 root root    0 Feb  8 11:18 notify_on_release
-rw-r--r--  1 root root    0 Feb  8 11:18 release_agent
drwxr-xr-x 68 root root    0 Feb  8 11:18 system.slice/
-rw-r--r--  1 root root    0 Feb  8 11:18 tasks
drwxr-xr-x  2 root root    0 Feb  8 11:18 test/
drwxr-xr-x  2 root root    0 Feb  8 11:18 user.slice/


# mount|grep cgroup|grep cpuacct
cgroup on /sys/fs/cgroup/cpu,cpuacct type cgroup (rw,nosuid,nodev,noexec,relatime,cpu,cpuacct)
new-cpu-cpuacct on /home/ubuntu/cgroup/cpu,cpuacct type cgroup (rw,relatime,cpu,cpuacct)


清理环境
#  umount new-cpu-cpuacct
```
从上面的实现可以看到新挂载点里面已经有了test目录，就像一个磁盘挂载到两个目录一样的效果，他们是同一个cgroup树


## 一个subsystem只能关联到一颗cgroup树
* 挂载一颗cgroup树时，可以指定多个subsystem与之关联，但一个subsystem只能关联到一颗cgroup树，
一旦关联并在这颗树上创建了子cgroup，subsystems和这棵cgroup树就成了一个整体，不能再重新组合。
由于已经将cpu,cpuacct和一颗cgroup树关联并且他们下面有子cgroup了，所以就不能单独的将cpu和另一颗cgroup树关联。
```
# mkdir cpu
# mount -t cgroup -o cpu new-cpu ./cpu
mount: new-cpu is already mounted or /home/ubuntu/cgroup/cpu busy

```

## none 挂载
cgroup生成了默认文件
```go
# mount -t cgroup -o none,name=test test ./test
# ll test/
-rw-r--r-- 1 root root    0 Feb  8 13:29 cgroup.clone_children
-rw-r--r-- 1 root root    0 Feb  8 13:29 cgroup.procs
-r--r--r-- 1 root root    0 Feb  8 13:29 cgroup.sane_behavior
-rw-r--r-- 1 root root    0 Feb  8 13:29 notify_on_release
-rw-r--r-- 1 root root    0 Feb  8 13:29 release_agent
-rw-r--r-- 1 root root    0 Feb  8 13:29 tasks 包含当前系统所以进程

# cd test && sudo mkdir aaaa
# ll aaaa/
-rw-r--r-- 1 root root 0 Feb  8 13:31 cgroup.clone_children
-rw-r--r-- 1 root root 0 Feb  8 13:31 cgroup.procs
-rw-r--r-- 1 root root 0 Feb  8 13:31 notify_on_release
-rw-r--r-- 1 root root 0 Feb  8 13:31 tasks  空
```
* cgroup.clone_children
  这个文件只对cpuset（subsystem）有影响，当该文件的内容为1时，新创建的cgroup将会继承父cgroup的配置，
  即从父cgroup里面拷贝配置文件来初始化新cgroup
  
* cgroup.procs
  当前cgroup中的所有进程ID，系统不保证ID是顺序排列的，且ID有可能重复
  
* notify_on_release
  该文件的内容为1时，当cgroup退出时（不再包含任何进程和子cgroup），将调用release_agent里面配置的命令。
  新cgroup被创建时将默认继承父cgroup的这项配置。  
* release_agent
  里面包含了cgroup退出时将会执行的命令，系统调用该命令时会将相应cgroup的相对路径当作参数传进去。 
  注意：这个文件只会存在于root cgroup下面，其他cgroup里面不会有这个文件。
  
* tasks
  当前cgroup中的所有线程ID，系统不保证ID是顺序排列的
    
    
## 进程管理的cgroup
```go
# cat /proc/$$/cgroup
13:name=test:/
11:devices:/user.slice
10:cpu,cpuacct:/user.slice
9:freezer:/
8:pids:/user.slice/user-1000.slice
7:hugetlb:/
6:memory:/user.slice
5:blkio:/user.slice
4:perf_event:/
3:cpuset:/
2:net_cls,net_prio:/
1:name=systemd:/user.slice/user-1000.slice/session-45.scope
```

## 加入cgroup
```
# sudo sh -c 'echo 1421 > cgroup.procs'
```