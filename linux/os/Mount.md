# MOUNT

mount命令的标准格式如下
```
mount -t type -o options device dir
```

device: 要挂载的设备（必填）。有些文件系统不需要指定具体的设备，这里可以随便填一个字符串,如tempfs、proc等

dir: 挂载到哪个目录（必填）

type： 文件系统类型（可选）。大部分情况下都不用指定该参数，系统都会自动检测到设备上的文件系统类型

options： 挂载参数（可选）。 
options一般分为两类，一类是Linux VFS所提供的通用参数，就是每个文件系统都可以使用这类参数，
另一类是每个文件系统自己支持的特有参数


## 挂载虚拟硬盘

```go
 dd if=/dev/zero bs=1M count=128 of=./vdisk.img
 mkfs.btrfs ./vdisk.img
 sudo mount ./vdisk.img /mnt/
 sudo touch /mnt/a
 ll /mnt/a
 umount /mnt
```

## 多设备到一个文件夹
在上面的基础上再创建一个设备挂载到/mnt下
```go
dd if=/dev/zero bs=1M count=128 of=./vdisk1.img
mkfs.btrfs ./vdisk1.img
先touch一个文件，然后查看文件
sudo mount ./vdisk1.img /mnt/
ls /mnt
```
结论：重复挂载到同一个目录会发生目录覆盖的现象，只能看到最后挂载的文件，如果卸载之后就可以看到上一层的文件了

## 一个设备挂载到多个文件
```go
mkdir test/mnt1
mkdir test/mnt2
mount vdisk.img test/mnt1
mount vdisk.img test/mnt2
touch /test/mnt1/x
ll  /test/mnt2
```
结论：可以在这两个目录下都可以看到这个文件，同一个设备挂载是可以共享文件的，但这是没有锁机制保障的