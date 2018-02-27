# 存储选型

首先看看官网对存储的选型的介绍：

尽量少使用btrfs和zfs，他们都属于需要复杂配置的文件系统

overlay2是首选，其次是overlay。这些都不需要额外的配置。overlay2是Docker CE的默认选择。

devicemapper是下一个，但是direct-lvm对于生产环境是需要的，因为loopback-lvm零配置的性能非常差。

aufs仅在Ubuntu和Debian上受支持，并且可能需要安装额外的软件包，而btrfs仅在SLES上受支持，而SLES仅受Docker EE支持

下面是根据操作系统官网给的建议：

Ubuntu Docker CE：aufs，devicemapper，overlay2（Ubuntu的14.04.4或更高版本，16.04或更高版本）overlay，zfs， vfs
Debian Docker CE：aufs，devicemapper，overlay2（Debian的弹力）， overlay，vfs
CentOS Docker CE：devicemapper， vfs
Fedora Docker CE：devicemapper，overlay2（Fedora 26或更高版本，实验），overlay（实验），vfs
如果可能的话，docker是推荐使用overlay2