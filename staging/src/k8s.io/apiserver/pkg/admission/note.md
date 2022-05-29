# 准入

## 概述

准入控制器是 K8s 资源进入 Etcd 前的最后一道防线。

```go
type Interface interface {
    Handles(operation Operation) bool
}

type MutationInterface interface {
	Interface
	Admit(ctx context.Context, a Attributes, o ObjectInterfaces) (err error)
}

type ValidationInterface interface {
	Interface
	Validate(ctx context.Context, a Attributes, o ObjectInterfaces) (err error)
}
```

通常，准入控制器会实现 MutationInterface 或者 ValidationInterface，或者二者都实现。

K8s 支持的准入控制器列表有两处：
* [plugin/pkg/admission](../../../../../plugin/pkg/admission) *24
* [apiserver/pkg/admission/plugin](../../pkg/admission/plugin) *3

## 举例

常见的准入控制器有：
* [MutatingAdmissionWebhook](./plugin/webhook/mutating/plugin.go)

通常以 webhook 方式实现。