# NodeAffinity Prefilter

## 概述

这个插件在这个扩展点做的事情是：
* 通过 GetRequiredNodeAffinity 函数获取一个 Pod 内的所有"强制"条件（写入 CycleState 中），包括：nodeSelector 与 NodeAffinity.RequiredDuringSchedulingIgnoredDuringExecution
* 通过读取 RequiredDuringSchedulingIgnoredDuringExecution 字段，初步筛选出可能是 feasible 的 Node