## 添加到一个新的ns
```go
#define _GNU_SOURCE
#include <fcntl.h>
#include <sched.h>
#include <stdlib.h>
#include <stdio.h>
#include <unistd.h>

#define NOT_OK_EXIT(code, msg); {if(code == -1){perror(msg); exit(-1);} }

int main(int argc, char *argv[])
{
    int fd, ret;

    if (argc < 2) {
        printf("%s /proc/PID/ns/FILE\n", argv[0]);
        return -1;
    }

    //获取namespace对应文件的描述符
    fd = open(argv[1], O_RDONLY);
    NOT_OK_EXIT(fd, "open");

    //执行完setns后，当前进程将加入指定的namespace
    //这里第二个参数为0，表示由系统自己检测fd对应的是哪种类型的namespace
    ret = setns(fd, 0);
    NOT_OK_EXIT(ret, "open");

    //用一个新的bash来替换掉当前子进程
    execlp("bash", "bash", (char *) NULL);

    return 0;
}

```
测试
```go
先看之前uts的实验，在新的uts空间内执行
# echo $$
6915

在新窗口执行
# gcc join.c -o namespace_join && ./namespace_join /proc/6915/ns/uts
# hostname
container001
# readlink /proc/$$/ns/uts
uts:[4026532253]
```
这样就加入到这个ns的uts里面了

既然能加入，自然也可以退出
```go
#define _GNU_SOURCE
#include <sched.h>
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>

#define NOT_OK_EXIT(code, msg); {if(code == -1){perror(msg); exit(-1);} }

static void usage(const char *pname)
{
    char usage[] = "Usage: %s [optins]\n"
                   "Options are:\n"
                   "    -i   unshare IPC namespace\n"
                   "    -m   unshare mount namespace\n"
                   "    -n   unshare network namespace\n"
                   "    -p   unshare PID namespace\n"
                   "    -u   unshare UTS namespace\n"
                   "    -U   unshare user namespace\n";
    printf(usage, pname);
    exit(0);
}

int main(int argc, char *argv[])
{
    int flags = 0, opt, ret;

    //解析命令行参数，用来决定退出哪个类型的namespace
    while ((opt = getopt(argc, argv, "imnpuUh")) != -1) {
        switch (opt) {
            case 'i': flags |= CLONE_NEWIPC;        break;
            case 'm': flags |= CLONE_NEWNS;         break;
            case 'n': flags |= CLONE_NEWNET;        break;
            case 'p': flags |= CLONE_NEWPID;        break;
            case 'u': flags |= CLONE_NEWUTS;        break;
            case 'U': flags |= CLONE_NEWUSER;       break;
            case 'h': usage(argv[0]);               break;
            default:  usage(argv[0]);
        }
    }

    if (flags == 0) {
        usage(argv[0]);
    }

    //执行完unshare函数后，当前进程就会退出当前的一个或多个类型的namespace,
    //然后进入到一个或多个新创建的不同类型的namespace
    ret = unshare(flags);
    NOT_OK_EXIT(ret, "unshare");

    //用一个新的bash来替换掉当前子进程
    execlp("bash", "bash", (char *) NULL);

    return 0;
}

```
在宿主机上测试
```go

gcc leave.c -o namespace_leave

# readlink /proc/$$/ns/uts
uts:[4026531838]

./namespace_leave -u

# readlink /proc/$$/ns/uts
uts:[4026532254]

```