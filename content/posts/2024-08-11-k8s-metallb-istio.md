+++

title = 'metallb-istio'
date = 2024-08-11T19:31:27+08:00
draft = false

tags = ["metallb-istio"]
categories = ["DevOps","k8s"]

+++
## 服务暴露



### 一、metallb

```shell
helm repo add metallb https://metallb.github.io/metallb
helm install metallb metallb/metallb --version=0.13.7 -n metallb-system --create-namespace


helm repo add serialt https://serialt.github.io/helm-charts
helm install metallb-config serialt/metallb-config --set "ipPoolRange=192.168.80.40-192.168.80.49" -n metallb-system  --version 0.0.1




  
```





**helm 测试：**

https://artifacthub.io/packages/helm/bitnami/nginx

```
$ helm repo add bitnami https://charts.bitnami.com/bitnami
$ helm install my-release bitnami/nginx
```

```shell
[root@localhost k3s]# kubectl get svc
NAME               TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)        AGE
kubernetes         ClusterIP      10.43.0.1       <none>          443/TCP        4h
my-release-nginx   LoadBalancer   10.43.100.152   192.168.8.200   80:30840/TCP   63m
```

```shell
[root@localhost k3s]# curl 192.168.8.200
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```



### 二、Ingress-nginx

```yaml
controller:
  name: controller
  image:
    ## Keep false as default for now!
    chroot: false
    registry: docker.io
    image: serialt/controller
    digest: 
    digestChroot:
  dnsPolicy: ClusterFirstWithHostNet
  hostNetwork: true
  ingressClassResource:
    # -- Name of the ingressClass
    name: nginx
    # -- Is this ingressClass enabled or not
    enabled: true
    # -- Is this the default ingressClass for the cluster
    default: true
    controllerValue: "k8s.io/ingress-nginx"
  publishService:  # hostNetwork 模式下设置为false，通过节点IP地址上报ingress status数据
    enabled: false
  # 是否需要处理不带 ingressClass 注解或者 ingressClassName 属性的 Ingress 对象
  # 设置为 true 会在控制器启动参数中新增一个 --watch-ingress-without-class 标注
  watchIngressWithoutClass: false
  kind: DaemonSet
#   tolerations:   # kubeadm 安装的集群默认情况下master是有污点，需要容忍这个污点才可以部署
#   - key: "node-role.kubernetes.io/master"
#     operator: "Equal"
#     effect: "NoSchedule"

#   nodeSelector:   # 固定到master1节点
#     kubernetes.io/hostname: master1
  service:  # HostNetwork 模式不需要创建service
    enabled: false
    
  admissionWebhooks: # 强烈建议开启 admission webhook
    enabled: true
    createSecretJob:
      resources:
        limits:
          cpu: 10m
          memory: 20Mi
        requests:
          cpu: 10m
          memory: 20Mi
    patchWebhookJob:
      resources:
        limits:
          cpu: 10m
          memory: 20Mi
        requests:
          cpu: 10m
          memory: 20Mi
    patch:
      enabled: true
      image:
        registry: docker.io
        image: serialt/kube-webhook-certgen
        digest:


defaultBackend:  # 配置默认后端
  enabled: true
  name: defaultbackend
  image:
    registry: docker.io
    # arm64 架构配置 serialt/defaultbackend-arm64
    image: serialt/defaultbackend-amd64
tcp: {}
#  8080: "default/example-tcp-svc:9000"

udp: {}
#  53: "kube-system/kube-dns:53"
```

```shell
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update

helm upgrade  --install ingress-nginx -n ingress-nginx --create-namespace   ingress-nginx/ingress-nginx --version 4.5.2 -f ingress-nginx.yaml 
```

