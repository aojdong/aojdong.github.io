# k8s服务使用源 IP

## 问题背景
k8s上有个暴露的`nginx`服务, 采用`锁IP`进行防暴力破解，该措施
由于获取到的IP不正确，锁的IP针对了所有IP来源，导致所有其他的用户使用其他IP也无法登录。

## 原因分析
`锁IP`的实现逻辑是使用的`nginx`的`remote_addr`字段进行缓存计数进行校验拦截，
但是对于k8s服务进行代理转发的服务而言，默认情况下`remote_addr`无法获取`原始IP`，
只会拿到k8s代理节点的IP;

针对这种情况，在k8s官方有给出[说明和解决的建议方法。](https://kubernetes.io/zh-cn/docs/tutorials/services/source-ip/)

## 解决办法
服务部署（svc）的配置中`externalTrafficPolicy`策略成`local`, 且`sessionAffinity`为`ClientIP`
```bash
kubectl patch svc -n xxxx xxxx  -p '{"spec":{"externalTrafficPolicy":"Local", "sessionAffinity": "ClientIP"}}'
```
> `service.spec.externalTrafficPolicy` 设置为 `Local`， `kube-proxy` 只会将代理请求代理到本地端点，而不会将流量转发到其他节点。 
> 这种方法保留了原始源 IP 地址。如果没有本地端点，则发送到该节点的数据包将被丢弃.   

> `SessionAffinity`：基于客户端IP地址进行会话保持的模式，即第1次将某个客户端发起的请求转发到后端的某个Pod上，之后从相同的客户端发起的请求都将被转发到后端相同的Pod上。

对于数据包丢弃可能导致找不到服务的问题，可以将服务应用部署使用`daemonset`, 改成多pod,  每一个master节点一个pod.
```yaml
apiVersion: v1
kind: Service
metadata:
  name: xxxx
  namespace: xxx
spec:
  externalTrafficPolicy: Local
  sessionAffinity: ClientIP
  selector:
    app: xxx
  ports:
    - port: xxx
      targetPort: xxx
  loadBalancerIP: xxx
  type: LoadBalancer
```

