## chroot

chroot，即 change root directory (更改 root 目录)。在 linux 系统中，系统默认的目录结构都是以 `/`，即是以根 (root) 开始的。
而在使用 chroot 之后，系统的目录结构将以指定的位置作为 `/` 位置。


* 增加了系统的安全性，限制了用户的权力；
在经过 chroot 之后，在新根下将访问不到旧系统的根目录结构和文件，这样就增强了系统的安全性。这个一般是在登录 (login) 前使用 chroot，以此达到用户不能访问一些特定的文件。

* 建立一个与原系统隔离的系统目录结构，方便用户的开发；
使用 chroot 后，系统读取的是新根下的目录和文件，这是一个与原系统根下文件不相关的目录结构。在这个新的环境中，可以用来测试软件的静态编译以及一些与系统不相关的独立开发。

* 切换系统的根目录位置，引导 Linux 系统启动以及急救系统等。
chroot 的作用就是切换系统的根位置，而这个作用最为明显的是在系统初始引导磁盘的处理过程中使用，从初始 RAM 磁盘 (initrd) 切换系统的根位置并执行真正的 init。另外，当系统出现一些问题时，我们也可以使用 chroot 来切换到一个临时的系统。


```go
sudo chroot . /busybox pwd
/
```
上面的例子通过chroot将当前的目录设置成根目录
如果是启动一个shell会更有意思，ok目录下面有busybox二进制
```go
sudo chroot ./ok  /busybox sh
/ # pwd
/

/ # /busybox ls /
busybox
```


pivot_root与chroot的一点区别：

（1）pivot_root会修改进程的根文件系统，chroot不会；
（2）pivot_root会修改进程的根目录、工作目录，chroot只会修改工作目录为根目录；
pivot_root主要是把整个系统切换到一个新的root目录，而移除对之前root文件系统的依赖，这样你就能够umount原先的root文件系统。
而chroot是针对某个进程，而系统的其它部分依旧运行于老的root目录。
