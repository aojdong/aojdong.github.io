# k8s dashboard、metrics-server和ingress部署

## k8s dashboard 介绍
- 通过dashboard能够直观了解Kubernetes集群中运行的资源对象
- 通过dashboard可以直接管理（创建、删除、重启等操作）资源对象

### 获取Kubernetes dashboard资源清单文件
```bash
wget https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml --no-check-certificate
```
修改 `recommended.yaml` 里面 `kubernetes-dashboard Service`为 `nodeport`
```yaml
kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
spec:
  type: NodePort
  ports:
    - port: 443
      targetPort: 8443
      nodePort: 30000  # 暴露端口
  selector:
    k8s-app: kubernetes-dashboard
```
kubectl部署
```bash
kubectl apply -f recommended.yaml
```

### 生成token
创建用户
```bash
# 创建 dashboard-admin 用户
kubectl create serviceaccount dashboard-admin -n kubernetes-dashboard
 
# 绑定 clusterrolebinding
kubectl create clusterrolebinding dashboard-admin-rb --clusterrole=cluster-admin --serviceaccount=kubernetes-dashboard:dashboard-admin
```
创建文件 `dashboard-admin-token.yaml`, 定义服务账户令牌，用于为 Kubernetes dashboard 提供访问权限。
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: dashboard-admin-secret
  namespace: kubernetes-dashboard
  annotations:
    kubernetes.io/service-account.name: dashboard-admin
type: kubernetes.io/service-account-token
```
创建并获取token, 访问页面时填入
```bash
kubectl apply -f dashboard-admin-token.yaml

kubectl describe secret dashboard-admin-secret -n kubernetes-dashboard
```

## 使用 `metrics-server` 实现主机资源监控

### 获取 `metrics-server` 资源清单文件
```bash
wget  https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.6.1/components.yaml --no-check-certificate
```
修改 `metrics-server` 资源清单文件
```yaml
# vim components.yaml
spec:
      containers:
      - args:
        - --cert-dir=/tmp
        - --secure-port=4443
        - --kubelet-preferred-address-types=InternalIP,InternalDNS,ExternalDNS,ExternalIP,Hostname
        - --kubelet-use-node-status-port
        - --metric-resolution=15s
        - --kubelet-insecure-tls 添加此行内容
```
部署
```bash
kubectl apply -f components.yaml
```

## 验证和访问
浏览器访问：`https://node-ip:30000/`


## ingress 部署
### 获取资源清单文件
```bash
wget https://raw.githubusercontent.com/kubernetes/ingress-nginx/nginx-0.30.0/deploy/static/mandatory.yaml --no-check-certificate
wget https://raw.githubusercontent.com/kubernetes/ingress-nginx/nginx-0.30.0/deploy/static/provider/baremetal/service-nodeport.yaml --no-check-certificate
```
> quay.io/kubernetes-ingress-controller/下的镜像拉取可以尝试使用七牛地址(quay-mirror.qiniu.com/kubernetes-ingress-controller)或者阿里云镜像地址(registry.cn-hangzhou.aliyuncs.com/google_containers/)替换

部署
```bash
kubectl apply -f mandatory.yaml 
kubectl apply -f service-nodeport.yaml 
```
查看Service
```bash
kubectl get svc -n ingress-nginx
```

### 指定需要进行代理的http服务，编写`ingress-http` 代理
```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-http
  namespace: default
spec:
  rules:
  - host: 域名
    http:
      paths:
      - path: /
        backend:
          serviceName: 服务名
          servicePort: 服务端口

```
启用ingress
```bash
kubectl apply -f ingress-http.yaml 
```

### 访问测试页面
直接使用IP进行访问时需要加上域名header：
```bash
curl  http://ingress-nginx-pod-ip -H 'host: 域名'
curl  http://node-ip:ingress-nginx-nodeport -H 'host: 域名'
```

使用域名访问时，需要配置`hosts`域名解析成node-ip
访问：http://域名:ingress-nginx-nodeport

## ingress 添加basic auth认证
### Htpasswd
htpasswd是Apache的Web服务器内置的工具,用于创建和更新储存用户名和用户基本认证的密码文件
### 生成用户密码文件
```bash
htpasswd -c auth admin
```
### 创建 `secret` 资源存储用户密码
```bash
kubectl create secret generic my-auth-secret --from-file=auth
```
### 修改 `ingress`资源
```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-http
  annotations:
    nginx.ingress.kubernetes.io/auth-type: basic
    nginx.ingress.kubernetes.io/auth-secret: my-auth-secret
    nginx.ingress.kubernetes.io/auth-realm: "Authentication Required"
...
```
