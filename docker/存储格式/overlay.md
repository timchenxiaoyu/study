# overlay

OverlayFS是一个类似于AUFS 的现代联合文件系统，但更快，实现更简单。Docker为OverlayFS提供了一个存储驱动程序。
* 注：OverlayFS是内核提供的文件系统，overlay和overlay2是docker提供的存储驱动

## 设置存储方式
编辑/etc/docker/daemon.json
```go
{
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ]
}
```

## overlay原理

OverlayFS将单个Linux主机上的两个目录合并成一个目录。这些目录被称为层，统一过程被称为联合挂载。OverlayFS关联的底层目录称为lowerdir，
对应的高层目录称为upperdir。合并过后统一视图称为merged。

下图是一个docker镜像和docke容器的分层图，docker镜像是lowdir，docker容器是upperdir。而统一的视图层是merged层
![](storage.md)
当镜像层和容器层都有相同的文件时候，使用容器层的文件，overlay驱动使用两层，这就意味着，如果是多层的镜像就无法使用了，替代的方案是：
每个镜像层都在/var/lib/docker/overlay目录下，通过硬链接的方式把下部的层关联起来
Docker1.10之后，镜像层ID和/var/lib/docker中的目录名不再一一对应。 
```go
$ docker pull ubuntu

Using default tag: latest
latest: Pulling from library/ubuntu

5ba4f30e5bea: Pull complete
9d7d19c9dc56: Pull complete
ac6ad7efd0f9: Pull complete
e7491a747824: Pull complete
a3ed95caeb02: Pull complete
Digest: sha256:46fb5d001b88ad904c5c732b086b596b92cfb4a4840a3abd0e35dbb6870585e4
Status: Downloaded newer image for ubuntu:latest
```
每个层都在/var/lib/docker/overlay/目录下面：
```go
$ ls -l /var/lib/docker/overlay/

total 20
drwx------ 3 root root 4096 Jun 20 16:11 38f3ed2eac129654acef11c32670b534670c3a06e483fce313d72e3e0a15baa8
drwx------ 3 root root 4096 Jun 20 16:11 55f1e14c361b90570df46371b20ce6d480c434981cbda5fd68c6ff61aa0a5358
drwx------ 3 root root 4096 Jun 20 16:11 824c8a961a4f5e8fe4f4243dab57c5be798e7fd195f6d88ab06aea92ba931654
drwx------ 3 root root 4096 Jun 20 16:11 ad0fe55125ebf599da124da175174a4b8c1878afe6907bf7c78570341f308461
drwx------ 3 root root 4096 Jun 20 16:11 edab9b5e5bf73f2997524eebeac1de4cf9c8b904fa8ad3ec43b3504196aa3801
```
overlay驱动程序只能使用一个较低的OverlayFS层，因此需要硬链接来实现多层图像。
镜像层目录包含对该镜像层唯一文件以及较低层通过硬链接的共享数据，这样做可以节省空间。
```go
$ ls -i /var/lib/docker/overlay/38f3ed2eac129654acef11c32670b534670c3a06e483fce313d72e3e0a15baa8/root/bin/ls

19793696 /var/lib/docker/overlay/38f3ed2eac129654acef11c32670b534670c3a06e483fce313d72e3e0a15baa8/root/bin/ls

$ ls -i /var/lib/docker/overlay/55f1e14c361b90570df46371b20ce6d480c434981cbda5fd68c6ff61aa0a5358/root/bin/ls

19793696 /var/lib/docker/overlay/55f1e14c361b90570df46371b20ce6d480c434981cbda5fd68c6ff61aa0a5358/root/bin/ls

```
两层的ls都是使用相同的ls二进制文件

容器层也位于/var/lib/docker/overlay/
```go
$ ls -l /var/lib/docker/overlay/<directory-of-running-container>

total 16
-rw-r--r-- 1 root root   64 Jun 20 16:39 lower-id
drwxr-xr-x 1 root root 4096 Jun 20 16:39 merged
drwxr-xr-x 4 root root 4096 Jun 20 16:39 upper
drwx------ 3 root root 4096 Jun 20 16:39 work

```
其中，该lower-id是容器镜像图顶层的ID，即Ove​​rlayFS lowerdir。
```go
$ cat /var/lib/docker/overlay/ec444863a55a9f1ca2df72223d459c5d940a721b2288ff86a3f27be28b53be6c/lower-id

55f1e14c361b90570df46371b20ce6d480c434981cbda5fd68c6ff61aa0a5358
```
该upper目录包含与OverlayFS相对应的容器的读写层的内容upperdir。

该merged目录是lowerdirand 的联合装载upperdir，它包含正在运行的容器内的文件系统的视图。

该work目录是OverlayFS内部的。

通过mount查看overlay文件的挂载
```go
$ mount | grep overlay

overlay on /var/lib/docker/overlay/ec444863a55a.../merged
type overlay (rw,relatime,lowerdir=/var/lib/docker/overlay/55f1e14c361b.../root,
upperdir=/var/lib/docker/overlay/ec444863a55a.../upper,
workdir=/var/lib/docker/overlay/ec444863a55a.../work)
```
在第二行的rw显示出overlay这层是可写的

## 文件操作
那么当我们读写文件的时候各层是怎样合作的呢？下面逐一来看，在此overlay和overlay2的行为是一样的
### 读
* 如果该文件在容器层不存在：文件不存在upperdir中，则从lowdir中读取
* 只在容器层存在：        文件只存在upperdir中，则直接从容器中读取改文件
* 文件同事存在容器曾和镜像层：容器层upperdir会覆盖镜像层lowdir中的文件

### 修改
* 首次写入： 改文件在upperdir中不存在，在overlay和overlay2中，执行copy_up操作，把文件从lowdir拷贝到upperdir。然后所以的更改都是在容器
层内修改副本的操作，由于overlayfs是文件级别的，不是块级别的，这就意味着即使文件只有很少的一点修改，也会产生的copy_up的行为，不过有两点需要
注意：
copy_up操作只发生在文件首次写入，以后都是只修改副本
overlayfs只适用两层，因此性能很好，查找搜索都更快

* 删除文件和目录： 当文件在容器被删除时，在容器层（upperdir）创建whiteout文件，镜像层的文件是不会被删除的，因为他们是只读的，但without文件
会阻止他们展现，当目录在容器内被删除时，在容器层（upperdir）一个不透明的目录，这个和上面whiteout原理一样，阻止用户继续访问，即便镜像层仍然
存在

### 重命名
rename(2)这个系统调用只在源和目标都在顶层，否则会报 error (“cross-device link not permitted”)

## 性能对比
使用overlay和overlay2驱动性能好于aufs和devicemapper，在实际生产环境overlay2的性能高于btrfs，但还有一些细节
* 页缓存：overlayfs支持页缓存共享，也就是说如果多个容器访问同一个文件，可以共享同一个页缓存。这使得overlay/overlay2驱动高效地利用了内存
* copy_up:aufs和overlayfs，由于第一次写入都会导致copy_up，尤其是大文件，会导致写延迟，以后的写入不会有问题。由于overlayfs层级
比aufs的多，所以ovelayfs的拷贝高于aufs
* inode限制：使用overlay存储驱动可能导致inode过度消耗，特别是当容器和镜像很多的情况下，所以建议使用overlay2.

### 高性能建议
* 使用高性能磁盘如：SSD
* 如果是搞IO:建议使用外挂高性能盘

但 overlay2驱动程序本身最多支持128个较低的OverlayFS层。此功能可为与层相关的Docker命令（如docker buildand）提供更好的性能docker commit，并在后备文件系统上占用更少的inode。