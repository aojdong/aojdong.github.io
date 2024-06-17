# ascend-operator学习

## 特性和功能
-	支持用户创建AscendJob。
-	支持用户创建client。
-	支持用户使用client对AscendJob进行增删改查
-	支持用户创建informer。

## 代码仓介绍
ascend-operator 代码仓内包含了两个go package
- `ascend-operator`: 提供AscendJob CRD 和 operator
- `ascend-operator-apis`: 提供AscendJob api 及其Clientsets, Listers、Informers。使用户能轻松对AscendJob进行CRUD操作。

在使用上，两个模块都是完全独立的。不存在互相引用。
集群中使用时，直接制作镜像，安装部署的就是 `ascend-operator` ，这里主要介绍的就是这块；
而 `ascend-operator-apis` 是提供给外部自定义客户端引入代码使用的。

## `ascend-operator` 核心代码指引：
`ascend-operator/pkg/api/v1/ascendjob_types.go` : AscendJob的CRD定义
`ascend-operator/pkg/api/v1/controllers/v1/ascendjob_controller.go` : AscendJob controller的核心逻辑


## 注释标记解释
`Kubebuilder` 利用一个叫做 `controller-gen` 的工具来生成公共的代码和` Kubernetes YAML` 文件。 
这些代码和配置的生成通过`Tags`（标签）来识别一个包是否需要生成代码及确定生成代码的方式，`Kubernetes`提供的`Tags`可以分为如下两种：
- `全局Tags`：定义在每个包的 `doc.go`文件中，对整个包中的类型自动生成代码。例如：`// +k8s:deepcopy-gen=package` 、`// +k8s:defaulter-gen=TypeMeta` 等等。
- `局部Tags`：定义在 Go 语言的类型声明上方，只对指定的类型自动生成代码。例如：`// +k8s:deepcopy-gen:interfaces=k8s.io/apimachinery/pkg/runtime.Object`


`Kubebuilder` 项目有两个 `make` 命令用到了 `controller-gen`：
- `make manifests` 用来生成 `Kubernetes` 对象的 `YAML` 文件，像`CustomResourceDefinitions`，`WebhookConfigurations` 和 `RBAC roles`。
- `make generate` 用来生成代码，像`runtime.Object/DeepCopy implementations`。

