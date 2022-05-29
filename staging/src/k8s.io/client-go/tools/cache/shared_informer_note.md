# SharedIndexInformer

## 概述

[SharedIndexInformer](shared_informer.go) 接口是对 [Controller](controller.go) 接口的再次抽象，并且内持一个带有索引的本地缓存（indexed local cache）

* 关于 Controller 接口的更多信息，请参考 [Controller](reflector_note.md)。
* 关于 Indexer 接口的更多信息，请参考 [Indexer](index_note.md)。

下面摘抄自 [sharedIndexInformer](shared_informer.go) 结构体的注释：

`sharedIndexInformer` implements SharedIndexInformer and has three main components:
* One is an indexed local cache, `indexer Indexer`. 
* The second main component is a Controller that pulls objects/notifications using the ListerWatcher and pushes them into a DeltaFIFO --- whose knownObjects is the informer's local cache --- while concurrently Popping Deltas values from that fifo and processing them with `sharedIndexInformer::HandleDeltas`.  Each invocation of HandleDeltas, which is done with the fifo's lock held, processes each Delta in turn. For each Delta this both updates the local cache and stuffs the relevant notification into the sharedProcessor.
* The third main component is that sharedProcessor, which is responsible for relaying those notifications to each of the informer's clients.

这里面有两点需要理解：
* 注意区分 [cache](store.go) 结构体与 [DeltaFIFO](delta_fifo.go) 结构体：
  * 前者的目的是利用缓存减少对 API Server 的压力，实现了 Store 接口甚至是 Indexer 接口
  * 后者的目的是为了生产者消费者解耦，实现了 Store 接口甚至是 Queue 接口
* 资源从 DeltaFIFO 弹出后，进入 `sharedIndexInformer.HandleDeltas()` 函数，在这个函数里有两个去处（以 Add 为例）：
  * cache：调用 `Store.Add(obj)`
  * sharedProcessor：调用 sharedProcessor.distribute() -> processorListener.addCh -> processorListener.nextCh -> ResourceEventHandler.OnAdd()