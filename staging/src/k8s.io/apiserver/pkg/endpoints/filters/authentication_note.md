# 认证

## 概述

认证流程通过装饰器模式包装到原始的 handler 处理函数外层。入口在 [WithAuthentication](./authentication.go) 函数中。 K8s 同时支持多种认证器，当启用多个时，会按照顺序依次调用认证器。每个认证器如何实现以及认证器的优先级均被隐藏到 [Request.AuthenticateRequest](../../authentication/authenticator/interfaces.go) 接口下。

当启用多个认证器时，它们被包装到 unionAuthRequestHandler 结构体中，再通过 [Request.AuthenticateRequest](../../authentication/authenticator/interfaces.go) 接口暴露出认证接口：

```go
func (authHandler *unionAuthRequestHandler) AuthenticateRequest(req *http.Request) (*authenticator.Response, bool, error) {
	...
	for _, currAuthRequestHandler := range authHandler.Handlers {
		resp, ok, err := currAuthRequestHandler.AuthenticateRequest(req)
		...
	}
	...
}
```

默认只要通过一个认证器即整个流程通过。

最后，将所有启用的认证器包装完成后，放到 GenericAPIServer.Handler.FullHandlerChain 中。
