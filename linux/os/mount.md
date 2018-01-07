## MOUNT


### bind
在做一些chroot的操作的时候,我们希望把当前的文件系统的一个目录(例如/dev)出现在chroot的目录下.
```go
mount --bind /dev $chrootdir/dev
```
这样,我们从chroot的目录和自己本身的文件系统的目录就都可以访问/dev目录.

可以创建一个挂载点,自己挂自己
```go
mount --bind foo foo
```

只读挂载
```go
mount --bind olddir newdir
mount -o remount,ro,bind olddir newdir
```
这样我再olddir新建的文件就可以在newdir看到了，但是是只读的，在newdir不能新建