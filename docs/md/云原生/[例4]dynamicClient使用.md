## [例4]dynamicClient
动态客户端，可以对任意的 Kubernetes 资源对象进行通用操作，包括 CRD。

下面例子实例化 `dynamicClient` 通过 `GVR` 操作集群资源

```go
package main

import (
	"context"
	"fmt"

	"call-k8s/util"
	corev1 "k8s.io/api/core/v1"
	metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
	"k8s.io/apimachinery/pkg/runtime"
	"k8s.io/apimachinery/pkg/runtime/schema"
	"k8s.io/client-go/dynamic"
	"k8s.io/client-go/tools/clientcmd"
)

func main() {
	// 加载配置文件 生成config对象
	config, err := clientcmd.BuildConfigFromFlags("", util.GetConfigPath())
	if err != nil {
		panic(any(err.Error()))
	}

	// 实例化客户端对象
	dynamicClient, err := dynamic.NewForConfig(config)
	if err != nil {
		panic(any(err.Error()))
	}

	// 配置需要的调用的GVR
	gvr := schema.GroupVersionResource{
		Group: "",  // 无名资源组不需要写  即core资源组
		Version: "v1",
		Resource: "pods",
	}

	// 发送请求且得到返回结果
	/*
	Resource 基于gvr生成了针对资源的客户端，即动态资源客户端 dynamicResourceClient
	Namespace 指定一个可操作的命名空间。同时Namespace、List都是dynamicResourceClient的方法
	List 通过restClient调用k8s APIServer接口 返回了Pod的数据，返回的数据格式是二进制的Json格式，然后通过一系列解析方法，转成了unstructured.UnstructuredList
	*/

	unSturctData, err := dynamicClient.Resource(gvr).Namespace("kube-system").List(context.TODO(), metav1.ListOptions{})
	if err != nil {
		panic(any(err.Error()))
	}

	// 转换成结构化的数据
	podList := corev1.PodList{}
	err = runtime.DefaultUnstructuredConverter.FromUnstructured(
		unSturctData.UnstructuredContent(),
		&podList)
	if err != nil {
		panic(any(err.Error()))
	}

	for _, item := range podList.Items {
		fmt.Println("namespace: ", item.Namespace, "name: ", item.Name)
	}
}
```
