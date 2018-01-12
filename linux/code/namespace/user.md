### USER
创建一个新的用户空间

```go
cat user.c
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
  printf("eUID = %ld;  eGID = %ld;  ",(long) geteuid(), (long) getegid());
  execv(child_args[0], child_args);
  return 1;
}

int main() {
  printf("程序开始: \n");
  int child_pid = clone(child_main, child_stack+STACK_SIZE,
            CLONE_NEWUSER | SIGCHLD, NULL);
  waitpid(child_pid, NULL, 0);
  printf("已退出\n");
  return 0;
}

```
测试如下
```go
root@tim:~# gcc -Wall user.c -o user.o && ./user.o
程序开始: 
在子进程中!
nobody@tim:~$ 
nobody@tim:~$ 
nobody@tim:~$ id -u
65534
nobody@tim:~$ id -g
65534
nobody@tim:~$ 
nobody@tim:~$ exit
exit
已退出
root@tim:~# id -u
0
root@tim:~# id -g
0

```