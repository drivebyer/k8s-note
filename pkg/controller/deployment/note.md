# deployment controller

## 初始化

deployment controller 需要以下几个组件：
* 能操作 K8s 的 clientset.Interface
* 关注的资源对应的 informer
* 用于解耦监听与处理逻辑的队列

## 关注的资源

deployment 需要 3 个 资源的 informer（每个 informer 有 2 类处理入口，分别是 Lister 与 ResourceEventHandler）：
* deployment
  * dLister
  * addDeployment：将 deployment 入队
  * updateDeployment：将新的 deployment 入队
  * deleteDeployment：将 deployment 入队
* replicaSet
  * rsLister
  * addReplicaSet：
    * 如果能找到 ReplicaSet 的 owner，则将 owner 入队
    * 如果找不到，则试图通过 label 找可能的 Deployment
  * updateReplicaSet
  * deleteReplicaSet
* pod
  * pLister
  * deletePod：当删除的是最后一个 Pod，则需要将 Deployment 重新入队

## syncDeployment

syncDeployment 从队列中取出 Deployment：
* 读取 Spec，应用一系列逻辑后，更新 Status
* Deployment 的 Pod 模版变更将触发 rollout 流程 

## 其它

* deleteDeployment：对于这个事件的处理仍然是将其入队，但是因为 deployment 已经从本地 Indexer 中删除，所以即是入队也不会有任何影响