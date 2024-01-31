# K8S二次开发

## CRD 和 operator

通常情况下，k8s提供了一些内置的资源类型，如Pod、Service、Deployment等，但是有时候用户需要定义自己的资源类型，
以便更好地管理他们的应用程序。Custom Resource可以让用户定义自己的API对象，并在k8s中使用它们。
这些自定义资源可以使用k8s API进行管理，就像其他内置资源一样。


### Resource、Custom Resource 和 Custom Controller
* **`Resource`** 是k8s中的基本对象类型，它代表了一个集群中的资源，例如Pod、Service、Deployment等。
* **`Custom Resource`** 是对k8s api的扩展，它允许用户定义自己的资源类型。这使得 k8s 更加模块化。
* `Custom Resource`可以通过**动态注册**的方式在运行中的集群内或出现或消失，集群管理员可以独立于集群更新`Custom Resource`。
* 一旦某`Custom Resource`被安装，用户**可以使用** `kubectl` **来创建和访问其中的对象**， 就像他们为 `Pod` 这种内置资源所做的一样。

* 就**`Custom Resource`**本身而言，它**只能用来存取结构化的数据**。
  当你将`Custom Resource`与`(Custom Controller)` 结合时， 定制资源就能够提供真正的**声明式 API`(Declarative API)`**。
* `Operator 模式`就是将`Custom Resource`与`(Custom Controller)` 相结合的

### 什么是CRD
如果 K8S 中的自带资源类型不足以满足业务需求，需要定制开发资源怎么办？`自定义资源(Custom Resource)`由此产生 。
那么，如何让k8s认识这些自定义的资源呢？`CRD(Custom Resource Definition)`就承担了一个说明书的角色，
让k8s 来认识这个自定义资源CR。

从 k8s 的用户角度来看，所有东西都叫`资源 Resource`, 除了常见内置资源之外，k8s 允许用户`自定义资源 Custom Resource`，
而 CRD 表示自定义资源的定义。

当你创建新的 CustomResourceDefinition（CRD）时，k8s API 服务器会为你所指定的每个版本**生成一个新的 RESTful 资源路径**。

基于 `CRD 对象`所创建的自定义资源可以是namespace **作用域**的，也可以是集群作用域的， 取决于 CRD 对象 `spec.scope` 字段的设置。


### CRD的由来
最早是谷歌提出`Third Party Resource`的概念，希望开发者以**插件化形式**扩展 K8s API 对象模型，以增强整个k8s的生态。
基于`Third Party Resource`这一概念，k8s 社区在 1.7 版本中提出了CRD 的概念。
k8s 1.7版本引入了OpenAPIv3Schema规范，用于定义k8s API的结构和验证规则。
随便打开一个CRD的YAML可以看到，其主体部分是使用 OpenAPI v3 schema 来描述CR的字段结构.


有了`CRD`之后，我们可以自由地增加各种内置资源平级的资源。原本很多之前只维护在软件内部的元数据，也可以被写入到k8s集群中。
这极大地拓宽了我们的想象力，比如作业、路由、账号等各种关联的资源都一股脑地放进集群里面去。

在各种自定义资源被放进去之后，就会有人问，这放进去是挺方便的，但是放进去就会生效吗？
是的，资源的生效就是 `Operator` 的功劳。

### 什么是 k8s Operator
OperatorFramework官网上对于 `Operator` 的解释：
`Operator`是指由人发出的，对k8s应用展开的操作部署、升级、扩缩容、卸载等管理k8s应用程序的方法。
即**operator应该就是个类似控制器的东西，里面含有一些运维操作**，
甚至是能够像运算符一样，让几种资源产生某种互动关系，一起协作完成复杂的工程行为。

`Operator` 其实并不是一个工具，而是为了解决一个问题而存在的一个思路，将特定于应用程序的操作知识编码到软件中，
利用功能强大的k8s抽象来正确地运行和管理应用程序。

### Operator的组成
`Operator` 是描述、部署和管理 k8s 应用的一套机制，从实现上来说，可以将其理解为 `CRD` 配合可选的 `webhook` 与 `controller` 
来实现用户业务逻辑，即 `operator = CRD + webhook + controller`。

- **CRD**: 允许用户自定义 k8s 资源，是一个资源定义；
- **webhook**: 它本质上是一种 HTTP 回调，会注册到 `apiserver` 上。在 `apiserver` 特定事件发生时，
  会查询已注册的 `webhook`，并把相应的消息转发过去。按照处理类型的不同，一般可以将其分为两类：
  一类可能会修改传入对象，称为 `mutating webhook`；一类则只读传入对象，称为 `validating webhook`。
- **controller**: 它会循环地处理工作队列，按照各自的逻辑把集群状态向预期状态推动。
  不同的 `controller` 处理的类型不同，比如 `replicaset controller` 关注的是副本数，会处理一些 Pod 相关的事件；
    - **工作队列**: controller 的核心组件。它会监控集群内的资源变化，并把相关的对象，包括它的动作与 key，
  例如 `Pod` 的一个 `Create` 动作，作为一个事件存储于该队列中；


### 常见的operator工作模式
![operator工作模式](https://img.zengzhiqi.top/2023/2-kubernetesercika/image_WAd8tQ38JN.png)
工作流程：

- 用户创建一个自定义资源定义 (`CRD`)；
- `apiserver` 根据自己注册的一个 `pass` 列表，把该 `CRD` 的请求转发给 `webhook`；
- `webhook` 一般会完成该 `CRD` 的缺省值设定和参数检验。
-  `webhook` 处理完之后，相应的 `CR` 会被写入数据库，返回给用户；

> 与此同时，`controller` 会在后台监测该自定义资源，按照业务逻辑，处理与该自定义资源相关联的特殊操作； 
上述处理一般会引起集群内的状态变化，`controller` 会监测这些关联的变化，把这些变化记录到 `CRD` 的状态中。

### Operator
不管是原生 `YAML / Helm` 还是 `Kustomize`，都是通过配置来搞定各类事情。然而 `CRD + Operator` 就不一样了，
它们让你直接接入 `apiserver`，作为 K8S 的一部分监听所有你关心的对象，并通过代码进行状态维持及管理。
因为 `CRD` 的开发是非常复杂的，除了业务逻辑之外，还需要做很多基础的工作，非常不便，
所以有了 `Operator` 的开发框架（常见的有 `KubeBuilder` 和 `Operator-SDK`），让开发人员专注于 `CRD` 的业务代码开发。


### operator开发框架：kuberbuilder和operator sdk
`operator sdk` 和 `kubebuilder` 都是为了用户方便创建和管理 `operator` 而生的脚手架项目。
`operator sdk` 在底层使用了 `kubebuilder` ，`operator sdk`的命令行工具底层实际是调用 `kubebuilder` 的命令行工具。

所以无论由`operator sdk`还是 `kubebuilder` 都是调用的 `controller-runtime` 接口。

`kuberbuilder` 是k8s官方提供的二次开发的框架。
- 提供脚手架工具初始化 CRDs 工程，自动生成 `boilerplate` 代码和配置；
- 提供代码库封装底层的 K8s `go-client`；

`operator sdk`则是由 `CoreOS` 开源，它是用于构建 k8s 原生应用的 SDK，它提供更高级别的 API、抽象和项目脚手架。

`kuberbuilder` 和 `operator sdk`两者之间大同小异，两者并不是竞争关系，`sdk` 相当于 `kubebuilder+`；

## 实践 kubebuilder

### 下载安装 kubebuilder
```bash
curl -L -o kubebuilder "https://go.kubebuilder.io/dl/latest/$(go env GOOS)/$(go env GOARCH)"
chmod +x kubebuilder && mv kubebuilder /usr/local/bin/
```
或者从 `https://go.kubebuilder.io/dl/latest/${os}/${arch} `下载安装包

### 创建一个项目
运行 `kubebuilder init` 命令来初始化一个新项目，该命令将下载 controller-runtime 二进制文件，并为我们准备好项目。如下
```bash
kubebuilder init --repo myk8soperator
```


`go.mod` 中引入了`client-go`、`controller-runtime` 和 `apimachinery`



- `cmd/main.go` 是项目入口，负责设置并运行管理器。
- `config/` 包含在 Kubernetes 中部署 Operator 的 manifest。
- `Dockerfile` 是用于构建管理器镜像的容器文件。

### <span id="creat_api">创建 API</span>
运行`kubebuilder create api` 命令创建一个新的 API（组/版本）并在其上 `ajd/v1` 创建新的 Kind：`Home`
```bash
kubebuilder create api --group ajd --version v1 --kind Home
```
`Create Resource [y/n]` 和 `Create Controller [y/n]` 按 `y`，那么这将自动生成两个文件：
- `api/v1/home_types.go`： 该文件中定义相关 API
- `internal/controllers/home_controller.go`： 该文件是当前Kind(CRD) 的协调业务逻辑的文件


> 注意：每当修改了CRD或者Controller的代码后，都需要重新运行`make manifests`命令来生成新的清单文件，
> 以便更新部署到Kubernetes集群中的Operator

### 测试
#### 安装 `CRD` 到当前K8S集群：
需要在 k8S 控制面进行安装
```bash
make install
```
可以通过`kubectl get crd -o wide`查看到安装的 `CRD`

#### 启动 Operator Controller
```bash
make run
```
- 通过在本地运行 Operator 来加快开发和调试过程，而不必每次都构建和部署 Operator。
- 通过将 Operator 绑定到 K8S 集群中，可以测试其与 K8S API 的交互，并确保其在 K8S 中的行为与预期一致。
- 在开发过程中，`make run` 还可以自动重新加载Operator代码，以便在进行更改后立即查看其效果，从而提高开发效率。

#### 运行 `CRD` 

`kubectl apply -k config/samples/`

#### 卸载
```bash
make uninstall
make undeploy
```

### 开发工作
#### 设计API

在前面[创建一个API](#creat_api)时，生成了两个文件，其中 `api/v1/home_types.go` 就是用来定义 API 的。
- `meta/v1` 包含所有 Kubernetes 种类共有的元数据
- 所有序列化的字段必须是 驼峰式 ，所以我们使用的 json 标签需要遵循该格式, json 标签是必需的
- 数字类型只支持 `int32` 和 `int64` 类型，对于小数，使用 `resource.Quantity` 类型
- 时间类型使用`metav1.Time`，而不是 `metav1.Time`

`home_types.go` 文件主要就是定义 `Kind` 对象涉及到的结构体和 `init` 函数，主要有:
  - `HomeSpec`: `spec` 代表所期望的状态, 所以控制器的任何 “输入” 都会在这里
  - `HomeStatus`: `status` 表示实际观察到的状态。它包含了我们希望用户或其他控制器能够获得的任何信息
  - `Home`: 是一个根类型, 它描述了当前 Kind, 一般不做修改。
  - `HomeList`: 多个 Home 的容器。它是批量操作中使用的 Kind, 一般不做修改。
通过 `init` 函数调用 `Register` 将 go 类型添加到 API 组中。

`api/v1/` 目录下还有两个文件，都不需要编辑:
- `groupversion_info.go` : 包含了关于 group-version 的一些元数据，保持原样
- `zz_generated.deepcopy.go`： 自动生成， 包含了前述 `runtime.Object` 接口的自动实现，这些实现标记了代表 `Kinds` 的所有根类型
> `runtime.Object` 接口就是`home_types.go`文件 中使用的类似`// +kubebuilder`的注释语句

#### 控制器介绍
控制器是 Kubernetes 以及任何 Operator 的核心。

任务：确保对于任何给定的对象（包括集群状态，以及潜在的外部状态，如 Kubelet 的运行容器或云提供商的负载均衡器）的实际状态与对象的期望状态相匹配。
每个控制器只专注于一个根 `Kind`，但可能会与其他 `Kind` 交互。 我们把这个过程称为 `reconciling`。

在 `controller-runtime` 中，为特定 `Kind` 实现 `reconciling` 的逻辑被称为 `Reconciler`。 
`Reconciler` 接受一个对象的名称，并返回我们是否需要再次尝试。

前面[创建一个API](#creat_api)时，其中生成的 `internal/controllers/home_controller.go` 就是用来实现 `Reconciler` 的。
基本结构就是一个 `struct` 和两个 `func`: 
- `HomeReconciler`: 基本的 `reconciler` 结构
- `Reconcile`: 对单个对象进行调谐, 可以使用 `client` 从缓存中获取 `Request` 对象
- `SetupWithManager`: 将 `reconciler` 添加到 `manager` 中，以便它在 `manager` 启动时一起启动。

#### main 函数
启动一个 `controller`
- 每组控制器都需要一个 `Scheme`，它提供 Kinds 与其对应的 Go 类型之间的映射。
- 实例化 `manager`，跟踪运行所有 `controller` ，以及设置 `Scheme` 和 API 服务器的客户端
- 运行 `manager` ，它会运行所有的 `controller` 和 `webhook`

#### 实现 `admission webhooks`
`admission webhooks` 包括 `Defaulter` 和 `Validator` 接口。
Kubebuilder 会帮你处理剩下的事情，：
- 创建 webhook 服务端。
- 确保服务端已添加到 manager 中。
- 为你的 webhooks 创建处理函数。
= 用路径在你的服务端中注册每个处理函数。

自动创建 `api/v1/home_webhook.go`, 且在 `main.go` 中搭建一个 `webhook` 函数的支架并用 `manager` 注册:
```bash
kubebuilder create webhook --group ajd --version v1 --kind Home --defaulting --programmatic-validation
```
文件
`api/v1/home_webhook.go` 里的构成如下：
- `SetupWebhookWithManager`: 将 webhook 和 manager 关联起来。
- `webhook.Defaulter`: 给 CRD 设置默认值的接口。webhook 会自动调用这个默认值。在 `Default()` 实现。
- `webhook.Validator`: 验证接口， `ValidateCreate()`, `ValidateUpdate()` 和 `ValidateDelete()` 方法期望在创建、更新和删除时 分别验证其接收者

#### 运行
测试 `controller` 只需要 `make install`。

想在本地运行 `webhooks`，必须为 `webhooks` 服务生成证书，并将它们放在正确的目录中（默认 /tmp/k8s-webhook-server/serving-certs/tls.{crt,key}）

设置环境变量禁用 `webhooks`, `make run ENABLE_WEBHOOKS=false`
