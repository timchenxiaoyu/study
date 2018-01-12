# 容器启动

## 1.init参数拼装

## 2.初始化rootfs
创建只读和读写层，并且挂载

## 3.挂载其它存储

## 4.启动init（parent进程）
读取cmd、切换root、挂载proc和tmpfs、执行cmd

## 设置cgroup
分为两步，先通过set创建，通过apply将pid写到tasks文件

## 网络连接





# 创建网络

## 创建网桥和POSTROUTEING

## 创建网络段