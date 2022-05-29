# Fit Prefilter/Filter

## Prefilter

### 概述

这个插件在这个扩展点做的事情是：从待调度的 Pod 中计算各个容器 ResourceRequirements.Resource 的总和。

### 详细

```go
func (f *Fit) PreFilter(ctx context.Context, cycleState *framework.CycleState, pod *v1.Pod) (*framework.PreFilterResult, *framework.Status) {
	cycleState.Write(preFilterStateKey, computePodResourceRequest(pod))
	return nil, nil
}
```

Fit 插件在扩展点 Prefilter 不会影响 Pod 的调度，但是会计算 Pod 的资源需求（），并将其写入 cycleState 中。

资源计算使用 [computePodResourceRequest](./fit.go) 函数。因为 Init 容器是串行，而普通容器是同时运行，所以该函数可以简单描述为：**max(range(init_containers), sum(normal_containers)) + Overhead**。

资源维度分为：
* CPU
* 内存
* 磁盘

# Filter