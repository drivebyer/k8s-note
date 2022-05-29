# 授权

## 概述

认证流程通过装饰器模式包装到原始的 handler 处理函数外层。入口在 [WithAuthorization](./authorization.go) 函数中。 K8s 同时支持多种授权器，当启用多个时，会按照顺序依次调用授权器。每个授权器需要实现下面的接口：

```go
type Authorizer interface {
	Authorize(ctx context.Context, a Attributes) (authorized Decision, reason string, err error)
}
```

```go
type RuleResolver interface {
    RulesFor(user user.Info, namespace string) ([]ResourceRuleInfo, []NonResourceRuleInfo, bool, error)
}
```
