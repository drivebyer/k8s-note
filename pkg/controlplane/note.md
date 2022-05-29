# api server(control plane)

## 创建流程

与 extension server 创建流程类似

关键的两个注册函数为：
* InstallLegacyAPI，路由为 localhost:8080/api/v1/pods
* InstallAPIs，路由为 localhost:8080/apis/apps/v1/deployments