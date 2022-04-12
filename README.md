# 有两种创建controller的方式
## 注册informer客户端的回调函数
kubernetes官方的sample-controller样例
[sample-controller](./sample-controller.md)

## 基于reconcile的方式
通过 kubebuilder 命令创建的 controller### main
[sample-kubebuilder](./sample-kubebuilder.md)

这两种方法底层都是使用watch方法（前者虽然是Watch&List方法，但是List中是空的），但是前者是使用注册informer的回调函数来处理不同的事件；后者使用manager这个interface来访问集群，并通过reconcile函数来维护资源的生命周期。
- 一个是通过事件驱动
- 一个是通过循环，自己控制生命周期