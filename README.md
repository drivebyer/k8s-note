# 笔记
> Latest update: 2022-05-29

## client-go

| 笔记                                                                                                   | 状态                 |
|------------------------------------------------------------------------------------------------------|--------------------|
| [Informer系列之Reflector](./staging/src/k8s.io/client-go/tools/cache/reflector_note.md)                 | :heavy_check_mark: |
| [Informer系列之Indexer](./staging/src/k8s.io/client-go/tools/cache/index_note.md)                       | :heavy_check_mark: |
| [Informer系列之SharedIndexInformer](./staging/src/k8s.io/client-go/tools/cache/shared_informer_note.md) | :heavy_check_mark: |
| [集群 leader 选举机制](./staging/src/k8s.io/client-go/tools/leaderelection/note.md)                        | :heavy_check_mark: |
| [事件管理机制]()                                                                                           | todo               |

## Controller Manager

| 笔记                                                           | 状态   |
|--------------------------------------------------------------|------|
| [Deployment Controller](./pkg/controller/deployment/note.md) | todo |

## API Server

| 笔记                                                                                          | 状态                 |
|---------------------------------------------------------------------------------------------|--------------------|
| [资源注册流程-核心服务](./pkg/api/legacyscheme/note.md)                                               | :heavy_check_mark: |
| [资源注册流程-扩展服务](./staging/src/k8s.io/apiextensions-apiserver/pkg/apiserver/note.md)           | :heavy_check_mark: |
| [资源注册流程-聚合服务](./staging/src/k8s.io/kube-aggregator/pkg/apiserver/scheme/note.md)️           | todo               |
| [配置与结构体创建流程](./cmd/kube-apiserver/app/note.md)                                              | :heavy_check_mark: |
| [Extension Server 创建流程](./staging/src/k8s.io/apiextensions-apiserver/pkg/apiserver/note.md) | :heavy_check_mark: |
| [API Server 创建流程](./pkg/controlplane/note.md)                                               | todo               |
| [Aggregator Server 创建流程](./staging/src/k8s.io/kube-aggregator/pkg/apiserver/note.md)        | todo               |
| [认证](./staging/src/k8s.io/apiserver/pkg/endpoints/filters/authentication_note.md)           | :heavy_check_mark: |
| [授权](./staging/src/k8s.io/apiserver/pkg/endpoints/filters/authorization_note.md)            | :heavy_check_mark: |
| [准入](./staging/src/k8s.io/apiserver/pkg/admission/note.md)                                  | :heavy_check_mark: |
| [对接 Etcd](./staging/src/k8s.io/apiserver/pkg/registry/note.md)                              | :heavy_check_mark: |

## Scheduler

* find all feasible nodes
* runs a set of functions to score the feasible Nodes
* picks a Node with the highest score
* binding: notifies the API server about this decision

|      | QueueSort                                                                         | PreFilter/Filter                                                                                                 | Filter | PreScore | Score | Normalize Score | Reserve | Permit | WaitOnPermit | PreBind | Bind | PostBind |
|------|-----------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------|--------|----------|-------|-----------------|---------|--------|--------------|---------|------|----------|
| 调用链路 | `NextPod -> MakeNextPodFunc -> SchedulingQueue.Pop -> activeQ.Pop -> ··· -> Less` | `scheduleOne -> SchedulePod -> findNodesThatFitPod -> RunPreFilterPlugins`                                       |        |          |       |                 |         |        |              |         |      |          |
|      | [PrioritySort](./pkg/scheduler/framework/plugins/queuesort/note.md)               | [Fit](./pkg/scheduler/framework/plugins/noderesources/fit_prefilter_filter_note.md)                              |        |          |       |                 |         |        |              |         |      |          |
|      |                                                                                   | [InterPodAffinity](./pkg/scheduler/framework/plugins/interpodaffinity/interpodaffinity_prefilter_filter_note.md) |        |          |       |                 |         |        |              |         |      |          |
|      |                                                                                   | [NodeAffinity](./pkg/scheduler/framework/plugins/nodeaffinity/node_affinity_prefilter_filter_note.md)            |        |          |       |                 |         |        |              |         |      |          |
|      |                                                                                   | [NodePorts](./pkg/scheduler/framework/plugins/nodeports/node_ports_prefilter_filter_note.md)                     |        |          |       |                 |         |        |              |         |      |          |
|      |                                                                                   | [PodTopologySpread]                                                                                              |        |          |       |                 |         |        |              |         |      |          |
|      |                                                                                   | [VolumeBinding](./pkg/scheduler/framework/plugins/volumebinding/prefilter_filter_note.md)                        |        |          |       |                 |         |        |              |         |      |          |
|      |                                                                                   | [VolumeRestrictions](./pkg/scheduler/framework/plugins/volumerestrictions/prefilter_filter_note.md)              |        |          |       |                 |         |        |              |         |      |          |
|      |                                                                                   |                                                                                                                  |        |          |       |                 |         |        |              |         |      |          |

## apimachinery

| 笔记                                                                   | 状态   |
|----------------------------------------------------------------------|------|
| [util/wait](./staging/src/k8s.io/apimachinery/pkg/util/wait/note.md) | todo |

## Tips

* GenericAPIServer.Handler.GoRestfulContainer 中注册了各个路由到 handler 的映射
* 注册路由的关键路经：InstallAPIGroup -> installAPIResources -> InstallREST -> Install -> registerResourceHandlers
* `kubectl proxy --port=8080` 后访问 http://127.0.0.1:8080/apis/apiextensions.k8s.io/v1
* ResourceVersion（利用 Etcd 中的 modifiedIndex 来实现） 有两个已知用处：1.客户端并发操作时实现乐观锁；2.ListWatch 时实现类似断点续传，防止数据丢失