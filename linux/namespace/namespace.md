Linux使用namespace来表示从不同进程角度所见的视图。 不同的namespace的进程看到的资源或者进程表是不一样的，相同namespace中的不同进程看到的是同样的资源。

Linux内核中就提供了这六种namespace隔离的系统调用，如下表所示。

| Namespace | 系统调用参数 | 隔离内容 |
| :------:| :------: | :------: |
|Cgroup|CLONE_NEWCGROUP|Cgroup的根目录 (since Linux 4.6)
| UTS | CLONE_NEWUTS | 主机名与域名 |
| IPC | CLONE_NEWIPC | 信号量、消息队列和共享内存 |
| PID | CLONE_NEWPID | 进程编号 |
| Network | CLONE_NEWNET | 网络设备、网络栈、端口等等 |
| Mount | CLONE_NEWNS | 挂载点（文件系统） |
| User | CLONE_NEWUSER | 用户和用户组 |

通过在clone系统调用里面传入不同系统调用参数就可以实现上面的不同隔离的功能，有意思的是，上面挂载点隔离叫做CLONE_NEWNS，搞得好像创建一个
namespace一样，这个其实mount隔离，只是历史原因导致的，因为当时设计的时候没有相关会有这么多namespace。

内核实现
```go
struct task_struct {
  ...
  /* namespaces */
  struct nsproxy *nsproxy;
  ...
}

struct nsproxy {
  atomic_t count;
  struct uts_namespace *uts_ns;
  struct ipc_namespace *ipc_ns;
  struct mnt_namespace *mnt_ns;
  struct pid_namespace *pid_ns_for_children;
  struct net       *net_ns;
  struct cgroup_namespace *cgroup_ns;
};
```
这个获取uts可以看到
```go
static inline struct new_utsname *utsname(void)
{
  //current指向当前进程的task结构体
  return &current->nsproxy->uts_ns->name;
}
```
处于不同UTS namespace中的进程，它task结构体里面的nsproxy->uts_ns所指向的结构体是不一样的，于是达到了隔离UTS的目的。
其他类型的namespace基本上也是差不多的原理。




















