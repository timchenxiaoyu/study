## PID
创建一个pid的ns，在这个里面pid讲重新开始计数。

```go
#define _GNU_SOURCE
#include <sys/types.h>
#include <sys/wait.h>
#include <stdio.h>
#include <sched.h>
#include <signal.h>
#include <unistd.h>

#define STACK_SIZE (1024 * 1024)

static char child_stack[STACK_SIZE];
char* const child_args[] = {
  "/bin/bash",
  NULL
};

int child_main(void* args) {
  printf("在子进程中!\n");
  execv(child_args[0], child_args);
  return 1;
}

int main() {
  printf("程序开始: \n");
  int child_pid = clone(child_main, child_stack+STACK_SIZE,
           CLONE_NEWPID | CLONE_NEWIPC | CLONE_NEWUTS 
           | SIGCHLD, NULL);
  waitpid(child_pid, NULL, 0);
  printf("已退出\n");
  return 0;
}

```
运行测试：
```go
gcc -Wall pid.c -o pid.o && ./pid.o
程序开始: 
在子进程中!
root@tim:~# echo $$
1
root@tim:~# exit
exit
已退出
```

### PID namespace中的init进程
在传统的UNIX系统中，PID为1的进程是init，地位非常特殊。他作为所有进程的父进程，维护一张进程表，不断检查进程的状态，
一旦有某个子进程因为程序错误成为了“孤儿”进程，init就会负责回收资源并结束这个子进程。所以在你要实现的容器中，
启动的第一个进程也需要实现类似init的功能，维护所有后续启动进程的运行状态。

### 信号与init进程
PID namespace中的init进程如此特殊，自然内核也为他赋予了特权——信号屏蔽。如果init中没有写处理某个信号的代码逻辑，
那么与init在同一个PID namespace下的进程（即使有超级权限）发送给它的该信号都会被屏蔽。这个功能的主要作用是防止init进程被误杀。

父节点中的进程发送的信号，如果不是SIGKILL（销毁进程）或SIGSTOP（暂停进程）也会被忽略。但如果发送SIGKILL或SIGSTOP，
子节点的init会强制执行（无法通过代码捕捉进行特殊处理），也就是说父节点中的进程有权终止子节点中的进程。
一旦init进程被销毁，同一PID namespace中的其他进程也会随之接收到SIGKILL信号而被销毁。理论上，该PID namespace自然也就不复存在了。
但是如果/proc/[pid]/ns/pid处于被挂载或者打开状态，namespace就会被保留下来。然而，
保留下来的namespace无法通过setns()或者fork()创建进程，所以实际上并没有什么作用。

我们常说，Docker一旦启动就有进程在运行，不存在不包含任何进程的Docker，也就是这个道理。


### 挂载proc文件系统
如果你在新的PID namespace中使用ps命令查看，看到的还是所有的进程，
因为与PID直接相关的/proc文件系统（procfs）没有挂载到与原/proc不同的位置。
所以如果你只想看到PID namespace本身应该看到的进程，需要重新挂载/proc，命令如下。
```go
root@tim:~# mount -t proc proc /proc
root@tim:~# ps
  PID TTY          TIME CMD
    1 pts/1    00:00:00 bash
   13 pts/1    00:00:00 ps
```
注意：因为此时我们没有进行mount namespace的隔离，所以这一步操作实际上已经影响了 root namespace的文件系统，
当你退出新建的PID namespace以后再执行ps a就会发现出错，再次执行mount -t proc proc /proc可以修复错误。