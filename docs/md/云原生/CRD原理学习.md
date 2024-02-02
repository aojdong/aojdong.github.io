# CRD原理学习

## apiserve 的[资源 url](https://kubernetes.io/zh-cn/docs/reference/using-api/api-concepts/#resource-uris)
`kube-apiserver` 的所有资源都归属于不同组，以 `API Group` 方式对外暴露,
所有资源类型要么是`集群作用域`的`（/apis/GROUP/VERSION/*）`，
要么是`名字空间作用域`的`（/apis/GROUP/VERSION/namespaces/NAMESPACE/*）`。 

`名字空间作用域`的资源类型会在其名字空间被删除时也被删除， 并且对该资源类型的访问是由定义在名字空间域中的授权检查来控制的。

> 注意： 核心资源使用 `/api` 而不是` /apis`，并且不包含 `GROUP` 路径段。

## 拓展 kube-apiserver API
目标是在其中增设一个资源组 `ajd.my.domain`，
`HTTP REST Path` 为`/apis/ajd.my.domain/`, 
且资源 `home.ajd.my.domain` 可被 `kubectl CRUD`。
即下图的 `Home` 部分:
![home_crd_des.png](../../_media/home_crd_des.png)

## 实现
最简单的方式是在集群中创建 CRD 对象
```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  # 固定格式 {kind_plural}.{group}，其中 homes 对应 spec.names.plural，ajd.my.domain对应 spec.group
  name: homes.ajd.my.domain 
spec:
  group: ajd.my.domain # 资源组，用在 URL 标识资源所属 Group
  names:
    kind: Home
    listKind: HomeList
    plural: homes  # 资源名复数，用在 URL 标识资源 Kind
    singular: home # 资源名单数，可用于 kubectl 匹配资源
    shortNames:   # 资源简称，可用于 kubectl 匹配资源
    - ho
  scope: Namespaced # Namespaced/Cluster
  versions:
  - name: v1
    served: true # 是否启用该版本，可使用该标识启动/禁用该版本 API
    storage: true # 唯一落存储版本，如果 CRD 含有多个版本，只能有一个版本被标识为 true
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              msg:
                type: string
              nobody:
                type: boolean
              stayOpen:
                type: array
                items:
                  type: string
    additionalPrinterColumns: # 声明 kubectl get 输出列，默认在 name 列之外额外输出 age 列，state 列
    - name: age
      jsonPath: .metadata.creationTimestamp
      type: date
    - name: message
      jsonPath: .spec.msg
      type: string
```


创建 crd/home.ajd.my.domain 之后，即可用 kubectl 直接操作 home 资源。操作体验和官方资源 Pod，Service 等相比并无二致.
```yaml
apiVersion: ajd.my.domain/v1  # 使用何种 API 版本创建资源
kind: Home                    # 欲创建资源类型
metadata:                     # 唯一标识资源的元数据，包括 name
  name: test-ajd              # namespace（默认值为 default）, UID（省略则在服务端自动生成) 等
spec:                         # 所期望的资源状态
  msg: hi there
```


> 这样就可以了？ 那么 pod 又在哪里呢？
> 是的，这样就是完成了，pod 也是一种资源类型，和我们这里的 CRD 其实并没啥关系。

## Kubernetes 的 API Discovery Discovery
`kubectl` 如何知道 `kube-apiserver` 存在某项资源 `ho`？
如何知道 `ho` 是资源 `home` 的简称？
如何知道 `homes` 属于哪个资源 `Group`？
如何知道 `homes` 支持什么操作？

这就涉及到 `Kubernetes` 的 `API Discovery` 机制。

`API Discovery` 不断监听 `CRD` 变化，负责将 `CRD` 声明同步转化为以下内存对象:
- APIGroupDiscovery (1.26+)
- APIGroup
- APIResourceList
并动态注册以下 API 返回对应资源
/apis/{group} 
/apis/{group}/{version}

在上面例子中，动态注册的路由就是 /apis/ajd.my.domain 和 /apis/ajd.my.domain/v1。

调整日志级别可以查看kubectl 的 请求过程：
```bash
kubectl get ho --cache-dir $(mktemp -d) -v 6
```
可以看到会逐个搜寻全部api信息, 以上疑问在这些结果中都能解决：
- GET /api
- GET /apis
- GET /apis/{group}
- GET /apis/{group}/{version}
- GET /api/v1

## CRD apiserver 内部
kube-apiserver 包含两个模块：
- `kube-apiserver` 模块：负责官方核心组 core/legacy (Pod, ConfigMap, Service 等) 和官方普通 Groups (apps, autoscaling, batch) 资源的处理
- `apiextensions-apiserver` 模块：后面1.6版本才开始引入，负责 CRD 及对应 Custom Resources 处理。

`apiextensions-apiserver` 模块由多个内置 controllers 驱动。
比较重要的控制器是 `controllers` 是 `DiscoveryController`（负责 `API Discovery`）、
`OpenAPI` 控制器（`OpenAPI Spec`），和 `customresource_handler`（负责资源的增删改查）。
其他的还有 CRD 状态字段更新、对应 `custom resource` 名称检查、CRD 删除清理等。


## OpenAPI 控制器
OpenAPI 控制器有两个：
- OpenAPIController v2 ：支持 OpenAPI Specification v2
- OpenAPIController v3 ：支持 OpenAPI Specification v3。
  
字段 schemas 对应 CRD 对象字段 .spec.versions[].schema， 
OpenAPIController v2 和 OpenAPIController v3 会监听 CRD 变化、自动生成 OpenAPISpec 并将其写入 kube-apiserver 模块 OpenAPI Spec，
由 kube-apiserver 路由 /openapi/v2 和 /openapi/v3 对外暴露。

OpenAPISpec 则类似使用说明书，它提供了使用 API 的规范：包括接收参数、数据格式约束、操作动词、返回数据类型等。

比如，在前面定义crd的yaml中schema字段新增msg字段的校验。
```yaml
schema:
  openAPIV3Schema:
    type: object
    properties:
      spec:
        type: object
        required: ["msg"] # 新增字段必输校验
        properties:
          msg:
            type: string
            maxLength: 10 # 新增长度上限校验
```


##  CRD 如何被持久化
kube-apiserver 通过委托模式串联 apiextensions-apiserver 模块 获得了 CRD 处理能力

HTTP 请求路由流程如下:
1. 先从 `kube-apiserver` 模块开始路由匹配，如果匹配核心组路由 `/api/**` 或者官方 `API Groups` 如 `/apis/apps/**`，`/apis/batch/**`，直接在本模块处理。
   如果不匹配，委托给 `apiextensions-apiserver` 处理
2. `apiextensions-apiserver` 模块 先看请求是否匹配路由 `/apis/apiextensions.k8s.io/**`，如果是则为 `CRD` 变更，直接在本模块处理；
   如果不匹配，再看是否匹配任意 `CRD` 对应的 Custom 路由 `/apis/{crd_group}/**`，如果任一匹配，直接在本模块处理。
   否则委托给 `notfoundhandler` 处理
3. notfoundhandler 返回 HTTP 404

回顾之前的内容，可以发现 `apiextensions-apiserver` 模块 自动为 `CRD` 生成了这些 `REST API`

- /apis/ajd.my.domain/v1/homes
- /apis/ajd.my.domain/v1/namespaces/{namespace}/homes
- /apis/ajd.my.domain/v1/namespaces/{namespace}/homes/{name}

原理是模块内的 `customresource_handler` 提供了 `/apis/{group}/{version}/({kind_plural} | namespaces/{namespace}/{kind_plural} | namespaces/{namespace}/{kind_plural}/{name})` 通配。
`customresource_handler` 实时读取所有 `CRD` 信息，负责 `custom resources` 的 `CRUD` 操作，并持有一个 `RESTStorage` (实现通常为 etcd)。
在 API 层业务（通用校验、解码转换、admission 等）成功后，`customresource_handler` 调用 `RESTStorage` 实施对象持久化。

路由 /apis 实际是 /apis/{group}/{version} 和 /apis/{group} 的聚合，由 kube-apiserver 的 kube-aggregator 模块提供

## 总结
### 前面内容
```
                      +--- DiscoveryController sync ---> HTTP Endpoints /apis/{group},/apis/{group}/{version}
                      |
CRD <---listwatch---> +--- OpenApiController sync ---> OpenAPI Spec in kube-apiserver module
                      |
                      +--- customresource_handler CRUDs
                           +---> /apis/{group}/{version}/{kind_plural}
                           +---> /apis/{group}/{version}/namespaces/{namespace}/{kind_plural}
                           +---> /apis/{group}/{version}/namespaces/{namespace}/{kind_plural}/{name}
```


### 从 Go 生成 CRD
手动维护 `CRD` 对象是笨办法，更好的方式是从 `Go Struct` 生成 `CRD` 声明。
`controller-tools` 项目的一个工具 `controller-gen` 提供了这种能力。

`controller-gen`可以将 Go Structs 中含有 JSON Tags 的字段均映射到了 OpenAPI Spec Properties，
字段注释映射到 description，字段类型映射到 type。
`kubebuilder` 工具中就有用上 `controller-gen`。

