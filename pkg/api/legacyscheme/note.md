# legacyscheme（核心资源）

这个包中的全局变量 `Schema` 保存了 API Server 所有支持的资源。`Schema` 的消费暂且不关心，本文只关心 `Schema` 的生成（即资源注册）。

API Server 中资源的注册，利用了 Go 语言里 `init()` 函数的特性。整个注册流程是隐式的，会在 API Server 执行 `main()` 函数前完成整个注册流程。

在 [import_known_versions.go](../../../pkg/controlplane/import_known_versions.go) 中，引入了 22 个资源组，分别 22 个 import：

```go
import (
	// These imports are the API groups the API server will support.
	_ "k8s.io/kubernetes/pkg/apis/admission/install"
	_ "k8s.io/kubernetes/pkg/apis/admissionregistration/install"
	_ "k8s.io/kubernetes/pkg/apis/apiserverinternal/install"
	_ "k8s.io/kubernetes/pkg/apis/apps/install"
	_ "k8s.io/kubernetes/pkg/apis/authentication/install"
	_ "k8s.io/kubernetes/pkg/apis/authorization/install"
	_ "k8s.io/kubernetes/pkg/apis/autoscaling/install"
	_ "k8s.io/kubernetes/pkg/apis/batch/install"
	_ "k8s.io/kubernetes/pkg/apis/certificates/install"
	_ "k8s.io/kubernetes/pkg/apis/coordination/install"
	_ "k8s.io/kubernetes/pkg/apis/core/install"
	_ "k8s.io/kubernetes/pkg/apis/discovery/install"
	_ "k8s.io/kubernetes/pkg/apis/events/install"
	_ "k8s.io/kubernetes/pkg/apis/extensions/install"
	_ "k8s.io/kubernetes/pkg/apis/flowcontrol/install"
	_ "k8s.io/kubernetes/pkg/apis/imagepolicy/install"
	_ "k8s.io/kubernetes/pkg/apis/networking/install"
	_ "k8s.io/kubernetes/pkg/apis/node/install"
	_ "k8s.io/kubernetes/pkg/apis/policy/install"
	_ "k8s.io/kubernetes/pkg/apis/rbac/install"
	_ "k8s.io/kubernetes/pkg/apis/scheduling/install"
	_ "k8s.io/kubernetes/pkg/apis/storage/install"
)
```

这 22 个 import 会依次触发各个包内的 `init()` 函数，在 `init()` 函数中，将当前包内的资源注册到 `Schema` 中。以熟悉的 core 组资源为例：

```go
func init() {
    Install(legacyscheme.Scheme)
}

func Install(scheme *runtime.Scheme) {
    utilruntime.Must(core.AddToScheme(scheme)) // 内部版本
    utilruntime.Must(v1.AddToScheme(scheme))   // 外部版本
    utilruntime.Must(scheme.SetVersionPriority(v1.SchemeGroupVersion))
}
```

`Install()` 函数中分别注册了内部版本和外部版本的资源，这两个版本的资源分别定义在：
* [pkg/apis/core](../../../pkg/apis/core/types.go): group_name = "", version = "__internal"
* [k8s.io/api/core/v1](../../../vendor/k8s.io/api/core/v1/types.go): group_name = "", version = "v1"

注：K8s 中核心组 core 的组名表示为空字符串。