## [例7]watch监视
用于监视Kubernetes API对象变化的组件，它会订阅Kubernetes API服务器上的对象，并在对象发生变化时触发相应的事件。

下面例子通过 `Watch` 监视指定 `namespace` 下的 `Deployments` 事件

```go
package main

import (
	"context"
	"fmt"

	"call-k8s/util"
	v1 "k8s.io/api/apps/v1"
	metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
	"k8s.io/apimachinery/pkg/watch"
	"k8s.io/client-go/kubernetes"
	"k8s.io/client-go/tools/clientcmd"
)

func main() {
	// 加载配置文件
	config, err := clientcmd.BuildConfigFromFlags("", util.GetConfigPath())
	if err != nil {
		panic(any(err.Error()))
	}

	// 实例化clientset对象
	clientset, err := kubernetes.NewForConfig(config)
	if err != nil {
		panic(any(err.Error()))
	}

	// 调用监听的方法
	w, err := clientset.AppsV1().Deployments("test-space").Watch(context.TODO(), metav1.ListOptions{})
	if err != nil {
		panic(any(err.Error()))
	}

	fmt.Println("start...")
	for {
		select {
		case e, ok := <-w.ResultChan():
			if !ok {
				fmt.Println("ResultChan closed")
				return
			}

			// fmt.Println("get event")
			if pod, ok := e.Object.(*v1.Deployment); ok {
				switch e.Type {
				case watch.Added:
					fmt.Printf("新增事件:%s/%s\n", pod.Namespace, pod.Name)
				case watch.Deleted:
					fmt.Printf("删除事件:%s/%s\n", pod.Namespace, pod.Name)
				case watch.Modified:
					fmt.Printf("更新事件:%s/%s\n", pod.Namespace, pod.Name)
				default:
					fmt.Printf("%s事件:%s\n", e.Type, pod.Name)

				}
			}
		}
	}
}
```
