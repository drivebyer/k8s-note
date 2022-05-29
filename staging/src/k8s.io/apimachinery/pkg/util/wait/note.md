# Wait

wait 库是一个提高代码健壮性的工具库。这个库里最核心的两个函数族是：
* BackoffUntil 
  * JitterUntil
  * JitterUntilWithContext 
  * Until
  * NonSlidingUntil
  * Forever
* WaitForWithContext

## BackoffUntil

用来每隔一段事件执行 fn，直到收到退出信号。时间间隔由 BackoffManager 接口控制。该接口有两种实现：
* exponentialBackoffManagerImpl
* jitteredBackoffManagerImpl
