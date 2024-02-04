## [例2]client-set
负责操作 Kubernetes 内置的资源对象，例如：Pod、Service等。

下面例子实例化 `clientset`, 通过 `CoreV1` 使用 `List` 查询集群资源

```go
package main

import (
	"context"
	"fmt"

	"call-k8s/util"
	metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
	"k8s.io/client-go/kubernetes"
	"k8s.io/client-go/tools/clientcmd"
)

func main() {
	// 加载配置文件
	config, err := clientcmd.BuildConfigFromFlags("", util.GetConfigPath())
	if err != nil {
		panic(any(err.Error()))
	}

	// 实例化clientset对象  只能操作内置资源对象
	clientset, err := kubernetes.NewForConfig(config)
	if err != nil {
		panic(any(err.Error()))
	}

	/*
	CoreV1 返回CoreV1Client实例
	Pods 调用了nwePods函数，该函数返回的PodInterface对象 PodInterface对象实现了Pods资源相关的全部方法，同时nwePods里面还将RESTClients实例对象赋值给了对应的Client属性
	List 内使用RESTClient与k8s APIServer进行交互
	*/
	pods, err := clientset.
		CoreV1().  // 返回CoreV1Client实例
		Pods("kube-system"). // 指定查询的资源以及指定资源的namespace，namespace为空则表示所有namespace
		List(context.TODO(), metav1.ListOptions{}) // 使用RESTClient与k8s APIServer进行交互
	if err != nil {
		panic(any(err.Error()))
	}

	for _, item := range pods.Items {
		fmt.Println("namespace: ", item.Namespace, "name: ", item.Name)
	}
}
```
