


/proc/meminfo中
Cached表示page cache所占用的内存大小；
SwapCached表示位于磁盘交换区的页缓存page cache
所以实际页缓存page cache的容量 =  Cached + SwapCached



Page cache用于缓存文件里的数据，不仅包括普通的磁盘文件，还包括了tmpfs文件，tmpfs文件系统是将一部分内存空间模拟成文件系统，
由于背后并没有对应着磁盘，无法进行paging(换页)，只能进行swapping(交换)，在执行drop_cache操作的时候tmpfs对应的page cache并不会回收。

```go
# free  -g
              total        used        free      shared  buff/cache   available
Mem:             15           2          12           0           0          12
Swap:             1           1           0

# cat  /proc/meminfo
Buffers:           69004 kB
Cached:           709700 kB
SwapCached:       121288 kB
Active:          1797148 kB
Inactive:        1181988 kB
Active(anon):    1678188 kB
Inactive(anon):   787600 kB

# mount -t tmpfs -o size=3G none /mytmpfs/
此时没有任何变化
# free  -g
              total        used        free      shared  buff/cache   available
Mem:             15           3          10           0           0          10
Swap:             1           1           0


# dd if=/dev/zero of=/tttt/abc bs=1G count=2
# free  -g
              total        used        free      shared  buff/cache   available
Mem:             15           5           7           2           2           7
Swap:             1           1           0
可以看到shared和buff/cache都变成了2G

# cat  /proc/meminfo
Buffers:           69724 kB
Cached:          2808480 kB
SwapCached:       121340 kB
Active:          4891580 kB
Inactive:        3280208 kB
Active(anon):    4771284 kB
Inactive(anon):  2884808 kB
Active(file):     120296 kB
Inactive(file):   395400 kB


执行drop_caches，再观察free命令的"cached"值，发现刚才增加的2GB并未被回收：
$ sync
$ sudo sh -c 'echo 1 > /proc/sys/vm/drop_caches'
$ free

```
tmpfs占用的page cache是不能通过drop_caches操作回收的，tmpfs占用的page cache同时也算进了”shared”中，也就是说，被视为共享内存。

共享内存也和tmpfs一样，属于page cache，但又不能被drop_caches回收。这里所说的共享内存包括：

SysV shared memory
是通过shmget申请的共享内存，用”ipcs -m”或”cat /proc/sysvipc/shm”查看；
POSIX shared memory
是通过shm_open申请的共享内存，用”ls /dev/shm”查看；
shared anonymous mmap
通过mmap(…MAP_ANONYMOUS|MAP_SHARED…)申请的内存，可以用”pmap -x”或者”cat /proc/<PID>/maps”查看；
注：mmap调用参数如果不是MAP_ANONYMOUS|MAP_SHARED，则不属于tmpfs，
比如MAP_ANONYMOUS|MAP_PRIVATE根本不属于page cache而是属于AnonPages，MAP_SHARED属于普通文件，对应的page cache可以回写硬盘并回收。



用户进程的内存页分为两种：file-backed pages（与文件对应的内存页），和anonymous pages（匿名页），比如进程的代码、
映射的文件都是file-backed，而进程的堆、栈都是不与文件相对应的、就属于匿名页。file-backed pages在内存不足的时候可以直接写回对应
的硬盘文件里，称为page-out，不需要用到交换区(swap)；而anonymous pages在内存不足时就只能写到硬盘上的交换区(swap)里，称为swap-out。