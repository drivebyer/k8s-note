# Reflector

## 概述

正如 [Reflector](./reflector.go) 的释义，它像一面反射的镜子，**持续的**将 API Server 中的变化反射给用户。

因为 Reflector 直接与 API Server 这个 K8s 集群中流量最大的组件打交道，所以它必须保持「友好」。通过 `wait.BackoffUntil`，：

```go
func NewNamedReflector(name string, lw ListerWatcher, expectedType interface{}, store Store, resyncPeriod time.Duration) *Reflector {
	...
	r := &Reflector{
		...
		// We used to make the call every 1sec (1 QPS), the goal here is to achieve ~98% traffic reduction when
		// API server is not healthy. With these parameters, backoff will stop at [30,60) sec interval which is
		// 0.22 QPS. If we don't backoff for 2min, assume API server is healthy and we reset the backoff.
		backoffManager:         wait.NewExponentialBackoffManager(800*time.Millisecond, 30*time.Second, 2*time.Minute, 2.0, 1.0, realClock),
		...
	}
	...
}

func (r *Reflector) Run(stopCh <-chan struct{}) {
	wait.BackoffUntil(func() {
		r.ListAndWatch(stopCh)
	}, r.backoffManager, true, stopCh)
}
```

在核心逻辑 ListAndWatch 中，Reflector 向 API Server 发起调用，整体流程如下：
* 首先通过 List 接口获取全量数据（顺便得到资源版本），并将数据存储到 Reflector.store 中
* 然后通过 Watch 接口监听相应版本资源的变化，并将变化应用到 Reflector.store 中

## 再次抽象

通常情况下，Reflector 并不直接对外提供服务。将 Reflector 实现隐藏到 [Controller](./controller.go) 接口中，并通过 [Config](./controller.go) 结构体来配置该接口。

前面提到 Reflector 将监听的资源保存在 Reflector.store 中，这是**生产行为**。到了 Controller 这一层，可以配置**消费行为**。具体来说，是放松 [Store](./store.go) 接口，将其扩展为 [Queue](./fifo.go) 接口：

```go
type Queue interface {
	Store
	Pop(PopProcessFunc) (interface{}, error)
	AddIfNotPresent(interface{}) error
	HasSynced() bool
	Close()
}
```

Controller 每隔一秒从 Queue 中弹出一个资源，并交给 Config.Process 处理。

## 总结

* 通常不直接使用 Reflector，而是使用 Controller
* Controller 通过 Queue 将生产与消费解耦
* 通过 Controller 可以配置资源的消费者函数
* Controller 再次被包装成 SharedIndexInformer 接口，这个接口是最常用的