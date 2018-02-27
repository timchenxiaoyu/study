## bind

### 基本操作
bind mount会将源目录绑定到目的目录
在做一些chroot的操作的时候,我们希望把当前的文件系统的一个目录(例如/dev)出现在chroot的目录下.

```
mount --bind /dev $chrootdir/dev
```
这样,我们从chroot的目录和自己本身的文件系统的目录就都可以访问/dev目录.

```
tree test
test
├── mnt1
│   └── disk1
└── mnt2
    └── disk2
    
mount --bind test/mnt1/disk1 test/mnt2/disk2
    
touch test/mnt1/disk1/xxx   
 
 
tree test
test
├── mnt1
│   └── disk1
│       └── xxx
└── mnt2
    └── disk2
        └── xxx
    
    
# 如果是挂载父目录
 tree test
test
├── mnt1
│   └── disk1
│       └── xxx
└── mnt2
    └── disk2
 
 mount --bind test/mnt1 test/mnt2
 
 test
 ├── mnt1
 │   └── disk1
 │       └── xxx
 └── mnt2
     └── disk1
         └── xxx
             
```

可以创建一个挂载点,自己挂自己
```go
mount --bind foo foo
```

## 只读挂载
```go
mount --bind olddir newdir
mount -o remount,ro,bind olddir newdir
或者直接挂载
mount -o ro,bind olddir newdir
```

eg
```
mount --bind -o bind,ro  test/mnt1 test/mnt2
tree test
test
├── mnt1
│   └── disk1
│       ├── 333
│       ├── xxx
│       └── yyy
└── mnt2
    └── disk1
        ├── 333
        ├── xxx
        └── yyy

# touch test/mnt1/disk1/444
ok
# touch test/mnt2/disk1/444
# touch: cannot touch 'test/mnt2/disk1/444': Read-only file system

```
这样我再olddir新建的文件就可以在newdir看到了，但是是只读的，在newdir不能新建
这里甚至可以把源和目标都设置成自己，这样就把自己设置成只读的目录了eg
```go
 # mount --bind -o bind,ro  test/mnt1 test/mnt1
 # touch test/mnt1/disk1/aaa
 touch: cannot touch 'test/mnt1/disk1/aaa': Read-only file system
```

## 单个文件挂载
我们也可以bind mount单个文件，这个功能尤其适合需要在不同版本配置文件之间切换的时候
```
# cat test/aa
11

# cat test/bb
22

# mount --bind test/aa test/bb

# cat test/bb
11

此时当我修改aa内容后，bb的内容不会一起改变，甚至是删除也不会影响
# rm -rf aa
# cat bb
11

# umount bb
# cat bb
22

```
