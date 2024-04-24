# kubevir 实战
## 介绍
KubeVirt是Red-Hat开源的，以容器方式运行的虚拟机项目。
是基于Kubernetes运行的，具体的来说是基于Kubernetes的CRD（自定义资源）增加虚拟机的运行和管理相关的资源。

特别是VM，VMI资源类型。也是说我们通过CRD进行增加关于虚拟机的资源类型，然后通过写YAML的形式来创建虚拟机等一系列的操作。

主要有两种资源类型：VM，VMI

- `VM`资源: 为群集内的 `VirtualMachineInstance` 提供管理功能，例如开机/关机/重启虚拟机，确保虚拟机实例的启动状态，与虚拟机实例是 1:1 的关系。
- `VMI`(VirtualMachineInstance)资源: 类似于 kubernetes Pod，是管理虚拟机的最小资源。一个 `VMI` 对象即表示一台正在运行的虚拟机实例，包含一个虚拟机所需要的各种配置。

通过以上资源，可以在Kubernetes上管理虚拟机。


## 步骤
### 安装依赖：
```
yum install -y qemu-kvm libvirt virt-install bridge-utils
```
### 检查虚拟化支持： 
```
virt-host-validate qemu
```
> 如果不支持，则需要让kubevirt使用软件模拟虚拟化的配置：\
kubectl create namespace kubevirt \
kubectl create configmap -n kubevirt kubevirt-config /\
--from-literal debug.useEmulation=true


### 安装：
```
export VERSION=v0.48.1
kubectl apply -f https://github.com/kubevirt/kubevirt/releases/download/${VERSION}/kubevirt-operator.yaml
kubectl apply -f https://github.com/kubevirt/kubevirt/releases/download/${VERSION}/kubevirt-cr.yaml
```

> 需要镜像：quay.io/kubevirt/virt-operator:v0.48.1

### `KubeVirt` 提供了一个命令行工具：`virtctl`
```
wget https://github.com/kubevirt/kubevirt/releases/download/${VERSION}/virtctl-${VERSION}-linux-amd64
mv virtctl-${VERSION}-linux-amd64 /usr/local/bin/virtctl
chmod +x /usr/local/bin/virtctl
virtctl -h     #help帮助命令
```
> 相关命令:\
停止：`virtctl stop vm`\
启动：`virtctl start vm`\
重启：`virtctl restart vm`\
进入：`virtctl console vm`

### 部署CDI:
`Containerized Data Importer（CDI）`项目提供了用于使 `PVC` 作为 `KubeVirt VM` 磁盘的功能。建议同时部署 CDI：
```
export VERSION=v1.59.0
kubectl apply -f https://github.com/kubevirt/containerized-data-importer/releases/download/$VERSION/cdi-operator.yaml
kubectl apply -f https://github.com/kubevirt/containerized-data-importer/releases/download/$VERSION/cdi-cr.yaml
```

### 下载官方的vm.yaml文件（来生成官方的虚拟机）
``` 
wget https://kubevirt.io/labs/manifests/vm.yaml --no-check-certificate
kubectl apply -f vm.yaml
kubectl get vm
virtctl start testvm
kubectl get pods
kubectl get vmi
```


### vir-vnc安装
  ```yaml
# cat virtvnc.yaml
# cat virtvnc.yaml 
apiVersion: v1
kind: ServiceAccount
metadata:
  name: virtvnc
  namespace: kubevirt
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: virtvnc
subjects:
  - kind: ServiceAccount
    name: virtvnc
    namespace: kubevirt
roleRef:
  kind: ClusterRole
  name: virtvnc
  apiGroup: rbac.authorization.k8s.io
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: virtvnc
rules:
  - apiGroups:
      - subresources.kubevirt.io
    resources:
      - virtualmachineinstances/console
      - virtualmachineinstances/vnc
    verbs:
      - get
  - apiGroups:
      - kubevirt.io
    resources:
      - virtualmachines
      - virtualmachineinstances
      - virtualmachineinstancepresets
      - virtualmachineinstancereplicasets
      - virtualmachineinstancemigrations
    verbs:
      - get
      - list
      - watch
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: virtvnc
  name: virtvnc
  namespace: kubevirt
spec:
  ports:
    - port: 8001
      protocol: TCP
      targetPort: 8001
      nodePort: 30026
  selector:
    app: virtvnc
  type: NodePort
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: virtvnc
  namespace: kubevirt
spec:
  replicas: 1
  selector:
    matchLabels:
      app: virtvnc
  template:
    metadata:
      labels:
        app: virtvnc
    spec:
      serviceAccountName: virtvnc
      nodeSelector:
        node-role.kubernetes.io/master: ''
      tolerations:
        - key: "node-role.kubernetes.io/master"
          operator: "Equal"
          value: ""
          effect: "NoSchedule"
      containers:
        - name: virtvnc
          image: quay.io/samblade/virtvnc:v0.1
          imagePullPolicy: IfNotPresent
          livenessProbe:
            httpGet:
              port: 8001
              path: /
              scheme: HTTP
            failureThreshold: 30
            initialDelaySeconds: 30
            periodSeconds: 10
            successThreshold: 1
            timeoutSeconds: 5
```
```
# 执行如下命令：
kubectl apply -f virtvnc.yaml
# 查看virtvnc服务端口：
kubectl get svc -n kubevirt virtvnc
 ```
