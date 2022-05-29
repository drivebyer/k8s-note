# Indexer

## 概述

Indexer 中的索引与数据库中的索引有以下相似之处：
* 通过 AddIndexer 添加索引，需要指定索引名与被索引的字段
* 使用索引时，需要指定索引名与被索引的字段
* 多个索引可以并存

理解 Indexer 的前提是理解下面 3 个概念：
* storage key：在底层 map 存储中的 key
* index name：即索引名称
* indexed value：从底层 map 存储中的 value（即存储对象） 中选取一个字段，并传入 IndexFunc 得到的值

Indexer 的本质是一个对象存储 map + 多个建立在这个对象存储上的索引。如下

![indexer](http://assets.processon.com/chart_image/62922ee1f346fb41eeb9c33d.png)

## 用例

Indexer 这个概念不太好理解，我们再来看看它是怎么用的。下面贴一个典型的用例：

添加索引建立函数：

```go
rsInformer.Informer().AddIndexers(cache.Indexers{
	controllerUIDIndex: func(obj interface{}) ([]string, error) {
		rs, ok := obj.(*apps.ReplicaSet)
		if !ok {
			return []string{}, nil
		}
		controllerRef := metav1.GetControllerOf(rs)
		if controllerRef == nil {
			return []string{}, nil
		}
		return []string{string(controllerRef.UID)}, nil
	},
})
```

通过索引建立函数名（index name） + 索引字段名（indexed value）查找符合条件的对象：

```go
objects, err := rsc.rsIndexer.ByIndex(controllerUIDIndex, string(controllerRef.UID))
if err != nil {
	utilruntime.HandleError(err)
	return nil
}
relatedRSs := make([]*apps.ReplicaSet, 0, len(objects))
for _, obj := range objects {
	relatedRSs = append(relatedRSs, obj.(*apps.ReplicaSet))
}
```

## 其它

* 向 Indexer 中添加对象时，不需要关心如何生成对象的 key，这个操作是由函数 `DeletionHandlingMetaNamespaceKeyFunc` 自动生成的