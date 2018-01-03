## MOUNT

Mount namespace通过隔离文件系统挂载点对隔离文件系统提供支持，它是历史上第一个Linux namespace，
所以它的标识位比较特殊，就是CLONE_NEWNS。
挂载传播（mount propagation）是指由一个挂载对象的状态变化导致的其它挂载对象的挂载与解除挂载动作的事件。
                       
共享关系（share relationship）。如果两个挂载对象具有共享关系，那么一个挂载对象中的挂载事件会传播到另一个挂载对象，反之亦然。
从属关系（slave relationship）。如果两个挂载对象形成从属关系，那么一个挂载对象中的挂载事件会传播到另一个挂载对象，但是反过来不行；在这种关系中，从属对象是事件的接收者


一个挂载状态可能为如下的其中一种：

共享挂载（shared）
从属挂载（slave）
共享/从属挂载（shared and slave）
私有挂载（private）
不可绑定挂载（unbindable）

传播事件的挂载对象称为共享挂载（shared mount）；接收传播事件的挂载对象称为从属挂载（slave mount）。
既不传播也不接收传播事件的挂载对象称为私有挂载（private mount）。
另一种特殊的挂载对象称为不可绑定的挂载（unbindable mount），它们与私有挂载相似，但是不允许执行绑定挂载，即创建mount namespace时这块文件对象不可被复制。

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
           CLONE_NEWNS | CLONE_NEWPID | CLONE_NEWIPC 
           | CLONE_NEWUTS | SIGCHLD, NULL);
  waitpid(child_pid, NULL, 0);
  printf("已退出\n");
  return 0;
}
```
此时执行mount -t proc proc /proc将不会出现pid时候的问题