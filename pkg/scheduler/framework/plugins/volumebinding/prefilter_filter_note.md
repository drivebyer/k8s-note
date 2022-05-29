## VolumeBind Prefilter Filter

## Prefilter

### 概述

在这个扩展点主要判断 PVC：
* 通过调用 API Server ，判断 Pod 定义中的 PVC 是否合法
* 如果 Pod 的 PVC 是立即绑定的，但是还未绑定成功，那么会将 Pod 重新放回队列，过一会再调度

新的概念：
* PVC 分为 2 类：
  * 普通 PVC
  * ephemeral inline volumes：指的是 Pod Spec 定义中的 Volume.Ephemeral 字段，因为这个 PVC 直接定义在 Pod 规约中，所以称为 inline
* isPVCFullyBound()：指 PVC 中申明的 PV 是否已经完全归属 PVC
* isVolumeBound()：指 Volume 中的 PVC 是否已经完全绑定

## Filter