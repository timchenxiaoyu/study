Linux内核中就提供了这六种namespace隔离的系统调用，如下表所示。

| Namespace | 系统调用参数 | 隔离内容 |
| :------:| :------: | :------: |
| UTS | CLONE_NEWUTS | 主机名与域名 |
| IPC | CLONE_NEWIPC | 信号量、消息队列和共享内存 |
| PID | CLONE_NEWPID | 进程编号 |
| Network | CLONE_NEWNET | 网络设备、网络栈、端口等等 |
| Mount | CLONE_NEWNS | 挂载点（文件系统） |
| User | CLONE_NEWUSER | 用户和用户组 |

通过在clone系统调用里面传入不同系统调用参数就可以实现上面的不同隔离的功能，有意思的是，上面挂载点隔离叫做CLONE_NEWNS，搞得好像创建一个
namespace一样，这个其实mount隔离，只是历史原因导致的，因为当时设计的时候没有相关会有这么多namespace。
























