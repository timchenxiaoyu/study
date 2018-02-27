# PID

制cgroup及其所有子孙cgroup里面能创建的总的task数量。

```
#  mount|grep pids
cgroup on /sys/fs/cgroup/pids type cgroup (rw,nosuid,nodev,noexec,relatime,pids)

# cd /sys/fs/cgroup/pids/
# mkdir test
# cd test/
# cat pids.current 
0
# cat pids.max 
max

```
pids.current: 表示当前cgroup及其所有子孙cgroup中现有的总的进程数量
pids.max: 当前cgroup及其所有子孙cgroup中所允许创建的总的最大进程数量，在根cgroup下没有这个文件

```go
# echo 1 > pids.max
# echo $$ > cgroup.procs
# cat cgroup.procs
bash: fork: retry: No child processes
bash: fork: retry: No child processes
bash: fork: retry: No child processes
bash: fork: retry: No child processes
bash: fork: Resource temporarily unavailable

打开一个新的窗口
$ cat cgroup.procs 
13895
```
有两种情况会使pids.current > pids.max
设置pids.max时，将其值设置的比pids.current小
pids.max只会在当前cgroup中的进程fork、clone的时候生效，将其他进程加入到当前cgroup时，不会检测pids.max

* 子cgroup会首先于父cgroup的限制，不能超过父cgroup的总额