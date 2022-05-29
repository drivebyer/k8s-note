# PrioritySort

## 实现

这个扩展点的实现比较简单：

```go
func (pl *PrioritySort) Less(pInfo1, pInfo2 *framework.QueuedPodInfo) bool {
	p1 := corev1helpers.PodPriority(pInfo1.Pod)
	p2 := corev1helpers.PodPriority(pInfo2.Pod)
	return (p1 > p2) || (p1 == p2 && pInfo1.Timestamp.Before(pInfo2.Timestamp))
}
```

正常情况使用 Pod 上的优先级排序，如果优先级相同，则使用 Pod 的入队（指调度器中的调度队列）时间戳排序。