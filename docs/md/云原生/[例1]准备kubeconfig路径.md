## [例1] 准备kubeconfig路径
获取当前程序的路径，并返回该路径下 kubeconfig 文件的路径

```go
package util

import (
	"fmt"
	"os"
	"os/exec"
	"path/filepath"
	"strings"
)

func GetConfigPath() string  {
    // 获取当前程序的绝对路径，保存在变量 file 中。
	file, _ := exec.LookPath(os.Args[0])
	// 转换为绝对路径，保存在变量 path 中
	path, _ := filepath.Abs(file)
	// 获取 path 中最后一个路径分隔符的位置，保存在变量 index 中。
	index := strings.LastIndex(path, string(os.PathSeparator))
	// 截取到最后一个路径分隔符的位置，得到程序所在目录的路径。
	path = path[:index]
	fmt.Println(path)
	// 拼接起来，得到 kubeconfig 文件的路径。
	return filepath.Join(path, "/kubeconfig")
}

```

通过 `"k8s.io/client-go/tools/clientcmd"` 下的 `clientcmd.BuildConfigFromFlags` 加载配置文件。

有了上面的配置文件，就可以通过配置文件实例化与` Kubernetes APIServer` 交互 `Client-Go` 客户端了：
分别是 RESTClient、DiscoveryClient、ClientSet、DynamicClient。
![Client-Go客户端了](https://fdevops.com/wp-content/uploads/2022/06/1656235939-image.png)

