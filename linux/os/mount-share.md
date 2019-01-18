## propagation type

每个挂载点都有一个propagation type标志, 由它来决定当一个挂载点的下面创建和移除挂载点的时候，
是否会传播到属于相同peer group的其他挂载点下去，也即同一个peer group里的其他的挂载点下面是不是也会创建和移除相应的挂载点.
现在有4种不同类型的propagation type：

* MS_SHARED: 从名字就可以看出，挂载信息会在同一个peer group的不同挂载点之间共享传播. 当一个挂载点下面添加或者删除挂载点的时候，
同一个peer group里的其他挂载点下面也会挂载和卸载同样的挂载点,默认情况下，如果父挂载点是MS_SHARED，那么子挂载点也是MS_SHARED的

* MS_PRIVATE: 跟上面的刚好相反，挂载信息根本就不共享，也即private的挂载点不会属于任何peer group

* MS_SLAVE: 跟名字一样，信息的传播是单向的，在同一个peer group里面，master的挂载点下面发生变化的时候，
slave的挂载点下面也跟着变化，但反之则不然，slave下发生变化的时候不会通知master，master不会发生变化。

* MS_UNBINDABLE: 这个和MS_PRIVATE相同，只是这种类型的挂载点不能作为bind mount的源，
主要用来防止递归嵌套情况的出现。这种类型不常见

## peer group
peer group就是一个或多个挂载点的集合，他们之间可以共享挂载信息。
目前在下面两种情况下会使两个挂载点属于同一个peer group（前提条件是挂载点的propagation type是shared）

* 利用mount --bind命令，将会使源和目的挂载点属于同一个peer group，当然前提条件是‘源’必须要是一个挂载点。

* 当创建新的mount namespace时，新namespace会拷贝一份老namespace的挂载点信息，
于是新的和老的namespace里面的相同挂载点就会属于同一个peer group。

```
# mkdir disks && cd disks
# dd if=/dev/zero bs=1M count=32 of=./disk1.img
# dd if=/dev/zero bs=1M count=32 of=./disk2.img
# dd if=/dev/zero bs=1M count=32 of=./disk3.img
# dd if=/dev/zero bs=1M count=32 of=./disk4.img

# mkfs.ext2 ./disk1.img
# mkfs.ext2 ./disk2.img
# mkfs.ext2 ./disk3.img
# mkfs.ext2 ./disk4.img

# mkdir disk1 disk2
# ls
disk1  disk1.img  disk2  disk2.img  disk3.img  disk4.img

#  cat /proc/self/mountinfo |grep / | sed 's/ - .*//'
18 23 0:17 / /sys rw,nosuid,nodev,noexec,relatime shared:7
19 23 0:4 / /proc rw,nosuid,nodev,noexec,relatime shared:12
20 23 0:6 / /dev rw,nosuid,relatime shared:2
21 20 0:14 / /dev/pts rw,nosuid,noexec,relatime shared:3
22 23 0:18 / /run rw,nosuid,noexec,relatime shared:5
23 0 253:1 / / rw,relatime shared:1
24 18 0:12 / /sys/kernel/security rw,nosuid,nodev,noexec,relatime shared:8
25 20 0:19 / /dev/shm rw,nosuid,nodev shared:4
26 22 0:20 / /run/lock rw,nosuid,nodev,noexec,relatime shared:6
27 18 0:21 / /sys/fs/cgroup ro,nosuid,nodev,noexec shared:9
28 27 0:22 / /sys/fs/cgroup/systemd rw,nosuid,nodev,noexec,relatime shared:10
29 18 0:23 / /sys/fs/pstore rw,nosuid,nodev,noexec,relatime shared:11
30 27 0:24 / /sys/fs/cgroup/net_cls,net_prio rw,nosuid,nodev,noexec,relatime shared:13
31 27 0:25 / /sys/fs/cgroup/cpuset rw,nosuid,nodev,noexec,relatime shared:14
32 27 0:26 / /sys/fs/cgroup/perf_event rw,nosuid,nodev,noexec,relatime shared:15
33 27 0:27 / /sys/fs/cgroup/blkio rw,nosuid,nodev,noexec,relatime shared:16
34 27 0:28 / /sys/fs/cgroup/memory rw,nosuid,nodev,noexec,relatime shared:17
35 27 0:29 / /sys/fs/cgroup/hugetlb rw,nosuid,nodev,noexec,relatime shared:18
36 27 0:30 / /sys/fs/cgroup/pids rw,nosuid,nodev,noexec,relatime shared:19
37 27 0:31 / /sys/fs/cgroup/freezer rw,nosuid,nodev,noexec,relatime shared:20
38 27 0:32 / /sys/fs/cgroup/cpu,cpuacct rw,nosuid,nodev,noexec,relatime shared:21
39 27 0:33 / /sys/fs/cgroup/devices rw,nosuid,nodev,noexec,relatime shared:22
40 19 0:34 / /proc/sys/fs/binfmt_misc rw,relatime shared:23
41 20 0:16 / /dev/mqueue rw,relatime shared:24
42 20 0:35 / /dev/hugepages rw,relatime shared:25
43 18 0:7 / /sys/kernel/debug rw,relatime shared:26
44 18 0:36 / /sys/fs/fuse/connections rw,relatime shared:27
90 23 0:38 / /var/lib/lxcfs rw,nosuid,nodev,relatime shared:72
92 22 0:39 / /run/user/1000 rw,nosuid,nodev,relatime shared:74

```

## 继承父挂载属性

可以看到/（根） 是shared，如果不是通过"sudo mount --make-shared /"可以改成shared。
默认情况下，子挂载点会继承父挂载点的propagation type
```
# mount --make-shared ./disk1.img ./disk1
因为根是共享的，上面的--make-shared可以省略

# mount --make-private ./disk2.img ./disk2
cat /proc/self/mountinfo |grep disk | sed 's/ - .*//'
94 23 7:0 / /home/ubuntu/testmount/disks/disk1 rw,relatime shared:76
96 23 7:1 / /home/ubuntu/testmount/disks/disk2 rw,relatime


# sudo mkdir ./disk1/disk3 ./disk2/disk4
# mount ./disk3.img ./disk1/disk3
# mount ./disk4.img ./disk2/disk4

# tree
.
├── disk1
│   ├── disk3
│   │   └── lost+found
│   └── lost+found
├── disk1.img
├── disk2
│   ├── disk4
│   │   └── lost+found
│   └── lost+found
├── disk2.img
├── disk3.img
└── disk4.img

#  cat /proc/self/mountinfo |grep disk| sed 's/ - .*//'
94 23 7:0 / /home/ubuntu/testmount/disks/disk1 rw,relatime shared:76
96 23 7:1 / /home/ubuntu/testmount/disks/disk2 rw,relatime
98 94 7:2 / /home/ubuntu/testmount/disks/disk1/disk3 rw,relatime shared:78
100 96 7:3 / /home/ubuntu/testmount/disks/disk2/disk4 rw,relatime

```
关于mountinfo在此还是要多说一点，第一列是挂载的ID,第二列是挂载的父ID，第三列是设备号major:minor
第四列是根目录，第五列是挂载目录的路径，第六列是挂载的mount option，第七列tag[:value]形式的可选字段
详细的看[官方文档](http://man7.org/linux/man-pages/man5/proc.5.html)
结论：上面的disk3继承了父挂载点属性shared，同理disk4也是private。

## share 和 private
```
# umount disk1/disk3
# umount disk2/disk4
# mkdir bind1 bind2
# mount --bind ./disk1 ./bind1
# mount --bind ./disk2 ./bind2
# tree
.
├── bind1
│   ├── disk3
│   └── lost+found
├── bind2
│   ├── disk4
│   └── lost+found
├── disk1
│   ├── disk3
│   └── lost+found
├── disk1.img
├── disk2
│   ├── disk4
│   └── lost+found
├── disk2.img
├── disk3.img
└── disk4.img

# cat /proc/self/mountinfo |grep disk| sed 's/ - .*//'
94 23 7:0 / /home/ubuntu/testmount/disks/disk1 rw,relatime shared:76
96 23 7:1 / /home/ubuntu/testmount/disks/disk2 rw,relatime
98 23 7:0 / /home/ubuntu/testmount/disks/bind1 rw,relatime shared:76
100 23 7:1 / /home/ubuntu/testmount/disks/bind2 rw,relatime shared:80


# mount ./disk3.img ./disk1/disk3
# mount ./disk4.img ./disk2/disk4
# tree
.
├── bind1
│   ├── disk3
│   │   └── lost+found
│   └── lost+found
├── bind2
│   ├── disk4
│   └── lost+found
├── disk1
│   ├── disk3
│   │   └── lost+found
│   └── lost+found
├── disk1.img
├── disk2
│   ├── disk4
│   │   └── lost+found
│   └── lost+found
├── disk2.img
├── disk3.img
└── disk4.img

```
从上面可以看出，由于disk1是shared，所以bind1也是共享并且在同一个peer group76，而disk2是private所以它下面bind的是一个新的group号80
重新挂载disk3和disk4后，这时，由于disk1/和bind1/属于同一个peer group，所以两个目录下的disk3目录都是
如下：
```
# touch bind1/disk3/aaa
# tree
.
├── bind1
│   ├── disk3
│   │   ├── aaa
│   │   └── lost+found
│   └── lost+found
├── bind2
│   ├── disk4
│   └── lost+found
├── disk1
│   ├── disk3
│   │   ├── aaa
│   │   └── lost+found
│   └── lost+found
├── disk1.img
├── disk2
│   ├── disk4
│   │   └── lost+found
│   └── lost+found
├── disk2.img
├── disk3.img
└── disk4.img

```
而在disk4里面创建的文件却不是
```
# touch disk2/disk4/xxx
# touch bind2/disk4/yyy
# tree
.
├── bind1
│   ├── disk3
│   │   ├── aaa
│   │   └── lost+found
│   └── lost+found
├── bind2
│   ├── disk4
│   │   └── yyy
│   └── lost+found
├── disk1
│   ├── disk3
│   │   ├── aaa
│   │   └── lost+found
│   └── lost+found
├── disk1.img
├── disk2
│   ├── disk4
│   │   ├── lost+found
│   │   └── xxx
│   └── lost+found
├── disk2.img
├── disk3.img
└── disk4.img

```
文件是不共享的
此时卸载disk3会导致bind1和disk1下的都会卸载
```
#  cat /proc/self/mountinfo |grep disk3 | sed 's/ - .*//'
102 94 7:2 / /home/ubuntu/testmount/disks/disk1/disk3 rw,relatime shared:82
# umount bind1/disk3
cat /proc/self/mountinfo |grep disk3 | sed 's/ - .*//'
空
```

## slave

```
# cat /proc/self/mountinfo |grep disk| sed 's/ - .*//'
94 23 7:0 / /home/ubuntu/testmount/disks/disk1 rw,relatime shared:76
# mount --bind --make-shared ./disk1 ./bind1
# mount --bind --make-slave ./bind1 ./bind2
# tree
.
├── bind1
│   ├── disk3
│   └── lost+found
├── bind2
│   ├── disk3
│   └── lost+found
├── disk1
│   ├── disk3
│   └── lost+found
├── disk1.img
├── disk2
├── disk2.img
├── disk3.img
└── disk4.img

# cat /proc/self/mountinfo |grep disk| sed 's/ - .*//'
94 23 7:0 / /home/ubuntu/testmount/disks/disk1 rw,relatime shared:76
96 23 7:0 / /home/ubuntu/testmount/disks/bind1 rw,relatime shared:76
98 23 7:0 / /home/ubuntu/testmount/disks/bind2 rw,relatime master:76

上面都是属于同一个peer group号
# mount ./disk3.img ./disk1/disk3/
# cat /proc/self/mountinfo |grep disk| sed 's/ - .*//'
94 23 7:0 / /home/ubuntu/testmount/disks/disk1 rw,relatime shared:76
96 23 7:0 / /home/ubuntu/testmount/disks/bind1 rw,relatime shared:76
98 23 7:0 / /home/ubuntu/testmount/disks/bind2 rw,relatime master:76
100 94 7:1 / /home/ubuntu/testmount/disks/disk1/disk3 rw,relatime shared:80
101 96 7:1 / /home/ubuntu/testmount/disks/bind1/disk3 rw,relatime shared:80
102 98 7:1 / /home/ubuntu/testmount/disks/bind2/disk3 rw,relatime master:80
```
此时添加到disk1/disk3目录下的内容会被同步到其它目录

```
# touch  disk1/disk3/bbb
# tree
.
├── bind1
│   ├── disk3
│   │   ├── bbb
│   │   └── lost+found
│   └── lost+found
├── bind2
│   ├── disk3
│   │   ├── bbb
│   │   └── lost+found
│   └── lost+found
├── disk1
│   ├── disk3
│   │   ├── bbb
│   │   └── lost+found
│   └── lost+found
├── disk1.img
├── disk2
├── disk2.img
├── disk3.img
└── disk4.img

# touch bind1/disk3/xxxx
# touch bind2/disk3/yyyy
# tree
.
├── bind1
│   ├── disk3
│   │   ├── bbb
│   │   ├── lost+found
│   │   ├── xxxx
│   │   └── yyyy
│   └── lost+found
├── bind2
│   ├── disk3
│   │   ├── bbb
│   │   ├── lost+found
│   │   ├── xxxx
│   │   └── yyyy
│   └── lost+found
├── disk1
│   ├── disk3
│   │   ├── bbb
│   │   ├── lost+found
│   │   ├── xxxx
│   │   └── yyyy
│   └── lost+found
├── disk1.img
├── disk2
├── disk2.img
├── disk3.img
└── disk4.img

```
此时卸载
```go
# umount ./disk1/disk3/
# cat /proc/self/mountinfo |grep disk| sed 's/ - .*//'
94 23 7:0 / /home/ubuntu/testmount/disks/disk1 rw,relatime shared:76
96 23 7:0 / /home/ubuntu/testmount/disks/bind1 rw,relatime shared:76
98 23 7:0 / /home/ubuntu/testmount/disks/bind2 rw,relatime master:76

# mount ./disk3.img ./bind2/disk3/
# cat /proc/self/mountinfo |grep disk| sed 's/ - .*//'
94 23 7:0 / /home/ubuntu/testmount/disks/disk1 rw,relatime shared:76
96 23 7:0 / /home/ubuntu/testmount/disks/bind1 rw,relatime shared:76
98 23 7:0 / /home/ubuntu/testmount/disks/bind2 rw,relatime master:76
100 98 7:1 / /home/ubuntu/testmount/disks/bind2/disk3 rw,relatime
```
上面挂载的bind2.后并没有出现之前的三个挂载点，只有一个，就是slave bind2下面挂载的目录，是不会反向传播到master上面的
那么自然在这个目录下创建的文件也不会被master所感知。

下面是[官网](https://www.kernel.org/doc/Documentation/filesystems/sharedsubtree.txt)的例子：


```
A slave mount is like a shared mount except that mount and umount events
	only propagate towards it.

	All slave mounts have a master mount which is a shared.

	Here is an example:

	Let's say /mnt has a mount which is shared.
	# mount --make-shared /mnt

	Let's bind mount /mnt to /tmp
	# mount --bind /mnt /tmp

	the new mount at /tmp becomes a shared mount and it is a replica of
	the mount at /mnt.

	Now let's make the mount at /tmp; a slave of /mnt
	# mount --make-slave /tmp

	let's mount /dev/sd0 on /mnt/a
	# mount /dev/sd0 /mnt/a

	#ls /mnt/a
	t1 t2 t3

	#ls /tmp/a
	t1 t2 t3

	Note the mount event has propagated to the mount at /tmp

	However let's see what happens if we mount something on the mount at /tmp

	# mount /dev/sd1 /tmp/b

	#ls /tmp/b
	s1 s2 s3

	#ls /mnt/b

	Note how the mount event has not propagated to the mount at
	/mnt


```



