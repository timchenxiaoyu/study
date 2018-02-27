# overlay2

overlay2原生支持128层，这提供docker build和docker commit更好的性能支持
在执行完docker pull ubuntu后，可以看到
```go
$ ls -l /var/lib/docker/overlay2

total 24
drwx------ 5 root root 4096 Jun 20 07:36 223c2864175491657d238e2664251df13b63adb8d050924fd1bfcdb278b866f7
drwx------ 3 root root 4096 Jun 20 07:36 3a36935c9df35472229c57f4a27105a136f5e4dbef0f87905b2e506e494e348b
drwx------ 5 root root 4096 Jun 20 07:36 4e9fa83caff3e8f4cc83693fa407a4a9fac9573deaf481506c102d484dd1e6a1
drwx------ 5 root root 4096 Jun 20 07:36 e8876a226237217ec61c4baf238a32992291d059fdac95ed6303bdff3f59cff5
drwx------ 5 root root 4096 Jun 20 07:36 eca1e4e1694283e001f200a667bb3cb40853cf2d1b12c29feda7422fed78afed
drwx------ 2 root root 4096 Jun 20 07:36 l
```
这个l目录是新加的，这里面都是软连接文件目录的简写标识，这个主要是为了避免mount时候页大小的限制
```go
$ ls -l /var/lib/docker/overlay2/l

total 20
lrwxrwxrwx 1 root root 72 Jun 20 07:36 6Y5IM2XC7TSNIJZZFLJCS6I4I4 -> ../3a36935c9df35472229c57f4a27105a136f5e4dbef0f87905b2e506e494e348b/diff
lrwxrwxrwx 1 root root 72 Jun 20 07:36 B3WWEFKBG3PLLV737KZFIASSW7 -> ../4e9fa83caff3e8f4cc83693fa407a4a9fac9573deaf481506c102d484dd1e6a1/diff
lrwxrwxrwx 1 root root 72 Jun 20 07:36 JEYMODZYFCZFYSDABYXD5MF6YO -> ../eca1e4e1694283e001f200a667bb3cb40853cf2d1b12c29feda7422fed78afed/diff
lrwxrwxrwx 1 root root 72 Jun 20 07:36 NFYKDW6APBCCUCTOUSYDH4DXAT -> ../223c2864175491657d238e2664251df13b63adb8d050924fd1bfcdb278b866f7/diff
lrwxrwxrwx 1 root root 72 Jun 20 07:36 UL2MW33MSE3Q5VYIKBRN4ZAGQP -> ../e8876a226237217ec61c4baf238a32992291d059fdac95ed6303bdff3f59cff5/diff
```
然后我们看看具体的目录下是什么,如果是最下面的是没有lower的
```go
 ls /var/lib/docker/overlay2/3a36935c9df35472229c57f4a27105a136f5e4dbef0f87905b2e506e494e348b/

diff  link

cat /var/lib/docker/overlay2/3a36935c9df35472229c57f4a27105a136f5e4dbef0f87905b2e506e494e348b/link

6Y5IM2XC7TSNIJZZFLJCS6I4I4


ls  /var/lib/docker/overlay2/3a36935c9df35472229c57f4a27105a136f5e4dbef0f87905b2e506e494e348b/diff

bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var

```
lower从第二次开始
```go
//最底层
cat 72dd847f8c4a40ce1762353c216d32e18db433b17c65e61ee1558758631fb59f/lower
cat: 72dd847f8c4a40ce1762353c216d32e18db433b17c65e61ee1558758631fb59f/lower: 没有那个文件或目录

//倒数第二次层
cat 91af527ebbb6357fb1694334be10105edd07432da7cb901ef17ecdaf28944442/lower
l/YSWCORVIDFAIEIAFPP5AWBJZ5G

//倒数第三次
cat 76bc3e1bdecdd1da6ecfea3086d7fecefa589e567da864fd5a4b910c04568bbb/lower
l/N7S5NM6TVQ4X7NFK7ROIQ6JOAP:l/YSWCORVIDFAIEIAFPP5AWBJZ5G

//倒数第四底层
cat 6d7bdb155539b21b411fe5a4b7ebd41a7bc92dfb5d0158b961622dee834e19d0/lower
l/Q7UBZ47OWOXEF4YL5POZBJ3UKY:l/N7S5NM6TVQ4X7NFK7ROIQ6JOAP:l/YSWCORVIDFAIEIAFPP5AWBJZ5G

//镜像最上层
cat c249bc61bf63b4f39b316b30f0dbe83bc6b9425f6fc92b28dd9b36bf80308f5e/lower
l/JF7WPJE6K6CN5A7SSJEYDWWWMA:l/Q7UBZ47OWOXEF4YL5POZBJ3UKY:l/N7S5NM6TVQ4X7NFK7ROIQ6JOAP:l/YSWCORVIDFAIEIAFPP5AWBJZ5G

```
通过lower标识了镜像的父层的分层关联关系

diff记录了本层的信息，
```go
ll  91af527ebbb6357fb1694334be10105edd07432da7cb901ef17ecdaf28944442/diff/

drwxr-xr-x 4 root root 4096 1月  26 02:23 etc
drwxr-xr-x 2 root root 4096 1月  26 02:23 sbin
drwxr-xr-x 3 root root 4096 1月  24 06:49 usr
drwxr-xr-x 3 root root 4096 1月  24 06:49 var
```
如果是容器层，还会多一个merge层，这个和overlay的merge的概念是一样的。譬如我启动一个容器，并在var目录下创建aaaa文件可以看到最上层的读写层
```go

ll a1d281675ce2eacb0617b989ae846e29b8890954b8917b2919fbd025f537d7a0/diff/var/

-rw-r--r-- 1 root root 0 1月  31 12:23 aaaa


ll a1d281675ce2eacb0617b989ae846e29b8890954b8917b2919fbd025f537d7a0/merged/var/

-rw-r--r-- 1 root root    0 1月  31 12:23 aaaa
drwxr-xr-x 2 root root 4096 4月  13 2016 backups
drwxr-xr-x 5 root root 4096 1月  24 06:49 cache
drwxr-xr-x 1 root root 4096 2月   5 2016 lib
drwxrwsr-x 2 root ftp  4096 4月  13 2016 local
lrwxrwxrwx 1 root root    9 1月  24 06:49 lock -> /run/lock
drwxr-xr-x 4 root root 4096 1月  24 06:49 log
drwxrwsr-x 2 root mem  4096 1月  24 06:49 mail
drwxr-xr-x 2 root root 4096 1月  24 06:49 opt
lrwxrwxrwx 1 root root    4 1月  24 06:49 run -> /run
drwxr-xr-x 2 root root 4096 1月  24 06:49 spool
drwxrwxrwt 2 root root 4096 1月  24 06:49 tmp

```

这里如果启动容器还有一点需要介绍，你会看到多了一个"读写层-init"，这个只读层，它的目的是为了初始化容器配置信息，譬如hostname等信息
```go
ll a1d281675ce2eacb0617b989ae846e29b8890954b8917b2919fbd025f537d7a0-init/diff/etc/hostname 
-rwxr-xr-x 1 root root 0 1月  31 12:21 a1d281675ce2eacb0617b989ae846e29b8890954b8917b2919fbd025f537d7a0-init/diff/etc/hostname
```

查看mount信息
```go
 mount|grep overlay
overlay on /var/lib/docker/overlay2/a1d281675ce2eacb0617b989ae846e29b8890954b8917b2919fbd025f537d7a0/merged type 
overlay (rw,relatime,lowerdir=/var/lib/docker/overlay2/l/EETCZ74DSQUEXSCTWSVYKD6RSA:
/var/lib/docker/overlay2/l/MOZ5Z5Y6HVYMH2C5H4HCP64VPX:/var/lib/docker/overlay2/l/JF7WPJE6K6CN5A7SSJEYDWWWMA:
/var/lib/docker/overlay2/l/Q7UBZ47OWOXEF4YL5POZBJ3UKY:/var/lib/docker/overlay2/l/N7S5NM6TVQ4X7NFK7ROIQ6JOAP:
/var/lib/docker/overlay2/l/YSWCORVIDFAIEIAFPP5AWBJZ5G,
upperdir=/var/lib/docker/overlay2/a1d281675ce2eacb0617b989ae846e29b8890954b8917b2919fbd025f537d7a0/diff,
workdir=/var/lib/docker/overlay2/a1d281675ce2eacb0617b989ae846e29b8890954b8917b2919fbd025f537d7a0/work)

```
上面lowdir（只读层），第一个是最上层，譬如EETCZ74DSQUEXSCTWSVYKD6RSA，这个顺序很重要