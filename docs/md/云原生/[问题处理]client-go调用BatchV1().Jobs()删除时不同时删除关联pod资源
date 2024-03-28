# `client-go`调用`BatchV1().Jobs()`删除时不同时删除关联pod资源

## 问题背景
使用`client-go`删除`job`资源时，发现该`job`关联的`pod`仍在继续运行。代码调用如下：
```go
err := client.Clientset.BatchV1().Jobs(namespace).Delete(context.Background(), jobName, metav1.DeleteOptions{})
```
## 原因分析
当你删除一个`job`时，如果不指定`PropagationPolicy`，`K8S`认会将关联的`pod`保留下来，以防止数据丢失。如果希望在删除`job`时同时删除关联的`pod`，需要在删除`job`时指定`PropagationPolicy`为`Background`或`Foreground`。

`PropagationPolicy`取值如下：
```
	// Orphans the dependents.
	DeletePropagationOrphan DeletionPropagation = "Orphan"
	// Deletes the object from the key-value store, the garbage collector will
	// delete the dependents in the background.
	DeletePropagationBackground DeletionPropagation = "Background"
	// The object exists in the key-value store until the garbage collector
	// deletes all the dependents whose ownerReference.blockOwnerDeletion=true
	// from the key-value store.  API sever will put the "foregroundDeletion"
	// finalizer on the object, and sets its deletionTimestamp.  This policy is
	// cascading, i.e., the dependents will be deleted with Foreground.
	DeletePropagationForeground DeletionPropagation = "Foreground"
```

## 解决办法
修改代码如下：
```go
propagationPolicy := metav1.DeletePropagationBackground
err := client.Clientset.BatchV1().Jobs(namespace).Delete(context.Background(), jobName, metav1.DeleteOptions{
		PropagationPolicy: &propagationPolicy,
	})
```

