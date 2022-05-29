# leaderelection

## 背景
当组件多副本（高可用）部署时，某些业务逻辑通常只需要一个副本执行，这就要求：
1. 副本间通过某种竞争机制，保证只有一个副本执行业务逻辑，这个副本称为 leader
2. 当 leader 副本挂掉时，被剩余副本感知，并保证在一定时间内，有一个副本能接管执行业务逻辑

## 实现
假设有一个组件 demo 使用了 leaderelection 机制，它部署了 3 副本。

关键的函数调用是：

```
func (le *LeaderElector) Run(ctx context.Context) {
	defer runtime.HandleCrash()
	defer func() {
		le.config.Callbacks.OnStoppedLeading()
	}()
	
	// 按照副本的启动顺序，除了一个副本外的其它副本会阻塞在这里
	if !le.acquire(ctx) {
		return // ctx signalled done
	}
	
	ctx, cancel := context.WithCancel(ctx)
	defer cancel()
	
	// 回调函数 OnStartedLeading 上面注册了只能运行在 leader 上的代码逻辑
	go le.config.Callbacks.OnStartedLeading(ctx)
	
	// 按照副本的启动顺序，第一个副本会走到这里，
	// renew 函数只会在两种情况下返回：
	// 	1. ctx 被取消
	// 	2. 更新租期失败
	//
	// 当 leader 副本从 Run 函数返回后，通常意味着当前副本已经不能提供高可用了。
	// 这时，剩余的副本会在 acquire 函数中拿到租期，并接管 leader 的角色
	le.renew(ctx)
}
```

整个 leaderelection 包采用了类似分布式锁的设计。锁是 K8s 中的资源，`acquire()` 与 `renew()` 背后实际上是针对 K8s 中的资源进行 CRUD 操作。这里的锁资源有以下几种类型：
- core 组下的 ConfigMap 资源
- core 组下的 Endpoints 资源
- coordination 组下的 Lease 资源
- 以上 3 中资源的组合类型

## 案例
在 K8s 中，使用 leaderelection 包的组件有 kube-controller-manager。我们通过介绍 kube-controller-manager 的使用方式来开始学习 leaderelection 的机制。

使用入口在 [leaderElectAndRun()](../../../../../cmd/kube-controller-manager/app/controllermanager.go)。可以看到它直接调用了 `RunOrDie()` 函数。

在 `leaderElectAndRun()` 函数中，最后一个语句是 `panic("unreachable")`。可见，正常情况下，一个副本称为 leader 角色后，不出什么意外，它会一直以 leader 角色运行下去。