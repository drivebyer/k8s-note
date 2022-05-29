# extension server

## 创建流程

流程分两大步骤：
* 将 extension server 中的资源与后端存储 Etcd 操作对上绑定
* 将 extension server 中的资源与前端路由绑定

```go
func (c completedConfig) New(delegationTarget genericapiserver.DelegationTarget) (*CustomResourceDefinitions, error) {
	genericServer, err := c.GenericConfig.New("apiextensions-apiserver", delegationTarget)
	
	...
	
	// 一个资源组对应一个 APIGroupInfo 
	// 根据下面的处理可以看出 apiGroupInfo.VersionedResourcesStorageMap 里面存储： 
	// v1beta1 -> customresourcedefinitions -> customResourceDefinitionStorage
	apiGroupInfo := genericapiserver.NewDefaultAPIGroupInfo(apiextensions.GroupName, Scheme, metav1.ParameterCodec, Codecs)
	if resource := "customresourcedefinitions"; apiResourceConfig.ResourceEnabled(v1.SchemeGroupVersion.WithResource(resource)) {
		customResourceDefinitionStorage, err := customresourcedefinition.NewREST(Scheme, c.GenericConfig.RESTOptionsGetter)
		storage[resource] = customResourceDefinitionStorage // 每个资源一个 Etcd 操作对象
		storage[resource+"/status"] = customresourcedefinition.NewStatusREST(Scheme, customResourceDefinitionStorage)
	}
	if len(storage) > 0 {
		apiGroupInfo.VersionedResourcesStorageMap[v1.SchemeGroupVersion.Version] = storage
	}
	
	// 注册路由
	// InstallAPIGroup -> installAPIResources -> InstallREST -> Install -> registerResourceHandlers
	s.GenericAPIServer.InstallAPIGroup(&apiGroupInfo)
	
	...
	
	return s, nil
}
```

## schema

注册在 [Install() 函数](../apis/apiextensions/install/install.go)中。