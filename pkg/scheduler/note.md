# Scheduler

## 问题：Pod 的调度过程

NextPod()

RunReservePluginsReserve(): 
虽然 scheduling cycle 是串行，但是 binding cycle 是并行的（因为这个过程是比较耗时的），所以，需要在并行前，先为当前需要调度的 pod 去预留一些资源。


RunPermitPlugins()：
* 实现一些 co-scheduling 的功能
* https://kubernetes.io/zh/docs/concepts/scheduling-eviction/scheduling-framework/#permit

调度器看到的必须是最新的实时的资源，这样才能保证调度决策是比较准确的。


VolumeBinding 结构体通过实现 PreBindPlugin 接口，来达到在 Pod bind Node 前，执行绑定 volume 的操作