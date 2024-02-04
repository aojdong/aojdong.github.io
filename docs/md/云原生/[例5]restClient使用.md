## [例5]restClient
最基础的客户端，主要是对 HTTP 请求进行了封装，支持 Json 和 Protobuf 格式的数据。

下面例子实例化 `restClient`，通过使用 `api` 和 `gv` 查询集群资源

```go
package main

import (
	"context"
	"fmt"

	"call-k8s/util"
	corev1 "k8s.io/api/core/v1"
	metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
	"k8s.io/client-go/kubernetes/scheme"
	"k8s.io/client-go/rest"
	"k8s.io/client-go/tools/clientcmd"
)

func main()  {
	// 加载配置文件
	config, err := clientcmd.BuildConfigFromFlags("", util.GetConfigPath())
	if err != nil {
		panic(any(err.Error()))
	}

	// 配置API 路径
	config.APIPath = "api"   // posd, /api/v1/pods

	// 配置分组版本  GV
	config.GroupVersion = &corev1.SchemeGroupVersion // 无名资源组 group: "", version: "v1"

	// 配置数据编解码工具
	config.NegotiatedSerializer = scheme.Codecs

	// 实例化 RESTClient 对象
	restClient, err := rest.RESTClientFor(config)
	if err != nil {
		panic(any(err.Error()))
	}

	// 定义接收返回值变量
	result := corev1.PodList{}

	// 跟APIServer交互
	/*
	Get, 定义请求方式  返回一个Request结构体对象  这个个Request结构体对象，就是构建访问APIServer请求用的
	依次执行了Namespace，Resource，VersionedParams， 构建与APIServer交互的参数
	Do方法通过request发起请求，然后通过transformResponse解析请求返回，并绑定到对应资源对象的结构体对象上，即此处的corev1.PodList对象
	request 先是检查了有没有可用的client, 之后就开始调用net/http包的功能了
	*/

	err = restClient.
		Get().  // 请求方式   返回一个Request结构体对象  就是构建访问APIServer请求用的
		Namespace("kube-system").  // 指定命名空间
		Resource("pods").  // 指定资源 传递资源名称
		VersionedParams(&metav1.ListOptions{}, scheme.ParameterCodec). // 参数及参数的序列化工具
		Do(context.TODO()). // 触发请求
		Into(&result)  // 把结果绑定到对应资源对象的结构体对象上
	if err != nil {
		panic(any(err.Error()))
	}
	for _, item := range result.Items {
		fmt.Println("namespace: ", item.Namespace, "name: ", item.Name)
	}
}

```
