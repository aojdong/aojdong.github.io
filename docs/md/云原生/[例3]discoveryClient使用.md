## [例3]discoveryClient
发现客户端，负责发现 APIServer 支持的资源组、资源版本和资源信息的。

下面例子实例化discoveryClient获取集群GVR

```go
package main

import (
	"fmt"

	"call-k8s/util"
	"k8s.io/apimachinery/pkg/runtime/schema"
	"k8s.io/client-go/discovery"
	"k8s.io/client-go/tools/clientcmd"
)

func main() {
	// 加载配置文件
	config, err := clientcmd.BuildConfigFromFlags("", util.GetConfigPath())
	if err != nil {
		panic(any(err.Error()))
	}

	// 实例化客户端
	discoveryClient, err := discovery.NewDiscoveryClientForConfig(config)
	if err != nil {
		panic(any(err.Error()))
	}

	// 发送请求 获取GVR资源
	/*
	ServerGroupsAndResources内
	ServerGroups 复制获取GV数据， 然后调用fetchGroupVersionResources，给这个方法传递GV参数。
	然后通过调用ServerResourcesForGroupVersion (restClient)方法获取GV对应的Resource资源数据。
	同时返回一个map[gv]resourceList的数据格式，最后处理map -> slice， 然后返回 gv slice
	*/
	_, apiResource, err := discoveryClient.ServerGroupsAndResources()
	if err != nil {
		panic(any(err.Error()))
	}

	for _, list := range apiResource {
		gv, err := schema.ParseGroupVersion(list.GroupVersion)
		if err != nil {
			panic(any(err.Error()))
		}
		for _, resource := range list.APIResources {
			fmt.Println("name: ", resource.Name, ", group: ", gv.Group, ", version: ", gv.Version)
		}
	}
}
```

上面例子使用 `discovery.NewDiscoveryClientForConfig` 实例化，若需要使用带本地缓存的可以使用 `disk.NewCachedDiscoveryClientForConfig`.
如下：
```go
...
import "k8s.io/client-go/discovery/cached/disk"

func main() {
    ...
    // 实例化客户端  本客户端负责将GVR数据缓存到本地文件中
    discoveryClient, err := disk.NewCachedDiscoveryClientForConfig(config, "./cache/discovery", "./cache/http",
        time.Minute*60)
    if err != nil {
        panic(any(err.Error()))
    }
    ...
}
```
