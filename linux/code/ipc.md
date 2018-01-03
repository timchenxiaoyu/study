## IPC

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
  int child_pid = clone(child_main, child_stack+STACK_SIZE,CLONE_NEWIPC | CLONE_NEWUTS | SIGCHLD, NULL);
  waitpid(child_pid, NULL, 0);
  printf("已退出\n");
  return 0;
}

```
在宿主机上面先创建
```go
ipcmk -Q
Message queue id: 0
root@tim# ipcs -q

------ Message Queues --------
key        msqid      owner      perms      used-bytes   messages    
0x0105b531 0          root       644        0            0       
```
然后编译启动
```go
root@tim:~# gcc -Wall ipc.c -o ipc.o && ./ipc.o
程序开始: 
在子进程中!
root@tim:~# ipcs -q

------ Message Queues --------
key        msqid      owner      perms      used-bytes   messages    

root@tim:~# exit
```