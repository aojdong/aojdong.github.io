## [例6]indexer索引查询
一个用于存储Kubernetes API对象的缓存, 对需要的缓存实现 `IndexFunc` , 提高索引效率

下面例子通过索引器函数查询数据

```go
package main

import (
	"errors"
	"fmt"

	v1 "k8s.io/api/core/v1"
	metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
	"k8s.io/client-go/tools/cache"
)

// NamespaceIndexFunc Namespace
func NamespaceIndexFunc(obj interface{}) ([]string, error) {
	pod, ok := obj.(*v1.Pod)
	if !ok {
		return nil, errors.New("类型不对")
	}
	return []string{pod.Namespace}, nil
}

// NodeNameIndexFunc NodeName
func NodeNameIndexFunc(obj interface{}) ([]string, error) {
	pod, ok := obj.(*v1.Pod)
	if !ok {
		return nil, errors.New("类型不对")
	}
	return []string{pod.Spec.NodeName}, nil
}

func main() {
	// 实例一个Indexer对象
	index := cache.NewIndexer(cache.MetaNamespaceKeyFunc, cache.Indexers{
		"namespace": NamespaceIndexFunc,
		"nodeName": NodeNameIndexFunc,
	})

	// 模拟数据
	pod1 := &v1.Pod{
		ObjectMeta: metav1.ObjectMeta{
			Name: "index-pod-1",
			Namespace: "default",
		},
		Spec: v1.PodSpec{NodeName: "node1"},
	}
	pod2 := &v1.Pod{
		ObjectMeta: metav1.ObjectMeta{
			Name: "index-pod-2",
			Namespace: "default",
		},
		Spec: v1.PodSpec{NodeName: "node2"},
	}
	pod3 := &v1.Pod{
		ObjectMeta: metav1.ObjectMeta{
			Name: "index-pod-3",
			Namespace: "kube-system",
		},
		Spec: v1.PodSpec{NodeName: "node3"},
	}

	// 数据写入到indexer中
	_ = index.Add(pod1)
	_ = index.Add(pod2)
	_ = index.Add(pod3)

	// 通过索引器函数查询数据
	fmt.Println("---------------namespace IndexFunc-----------")
	pods, err := index.ByIndex("namespace", "default")
	if err != nil {
		panic(any(err.Error()))
	}
	for _, pod := range pods {
		fmt.Println(pod.(*v1.Pod).Name)
	}
	fmt.Println("---------------nodeName IndexFunc-----------")
	pods, err = index.ByIndex("nodeName", "node2")
	if err != nil {
		panic(any(err.Error()))
	}
	for _, pod := range pods {
		fmt.Println(pod.(*v1.Pod).Name)
	}
}
```
