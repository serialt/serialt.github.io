+++

title = 'Ingress-Nginx'
date = 2024-08-11T19:27:27+08:00
draft = false

tags = ["ingress-nginx"]
categories = ["DevOps","k8s"]

+++
## Ingress-Nginx

kubernetes中暴露服务的三种方式：

* ClusterIP
* NodePort
* LoadBalance

​    这几种方式都是在service的维度提供的，service的作用体现在两个方面，对集群内部，它不断跟踪pod的变化，更新endpoint中对应pod的对象，提供了ip不断变化的pod的服务发现机制，对集群外部，他类似负载均衡器，可以在集群内外部对pod进行访问。但是，单独用service暴露服务的方式，在实际生产环境中不太合适：
ClusterIP的方式只能在集群内部访问。
NodePort方式的话，测试环境使用还行，当有几十上百的服务在集群中运行时，NodePort的端口管理是灾难。
LoadBalance方式受限于云平台，且通常在云平台部署ELB还需要额外的费用。

​    所幸k8s还提供了一种集群维度暴露服务的方式，也就是ingress。ingress可以简单理解为service的service，他通过独立的ingress对象来制定请求转发的规则，把请求路由到一个或多个service中。这样就把服务与请求规则解耦了，可以从业务维度统一考虑业务的暴露，而不用为每个service单独考虑。
举个例子，现在集群有api、文件存储、前端3个service，可以通过一个ingress对象来实现图中的请求转发：



### ingress 与ingress-controller

要理解ingress，需要区分两个概念，ingress和ingress-controller：

- ingress对象：
  指的是k8s中的一个api对象，一般用yaml配置。作用是定义请求如何转发到service的规则，可以理解为配置模板。
- ingress-controller：
  具体实现反向代理及负载均衡的程序，对ingress定义的规则进行解析，根据配置的规则来实现请求转发。

简单来说，ingress-controller才是负责具体转发的组件，通过各种方式将它暴露在集群入口，外部对集群的请求流量会先到ingress-controller，而ingress对象是用来告诉ingress-controller该如何转发请求，比如哪些域名哪些path要转发到哪些服务等等。

### ingress-controller

ingress-controller并不是k8s自带的组件，实际上ingress-controller只是一个统称，用户可以选择不同的ingress-controller实现，目前，由k8s维护的ingress-controller只有google云的GCE与ingress-nginx两个，其他还有很多第三方维护的ingress-controller，具体可以参考[官方文档](https://link.segmentfault.com/?url=https%3A%2F%2Fkubernetes.io%2Fdocs%2Fconcepts%2Fservices-networking%2Fingress-controllers%2F%23additional-controllers)。但是不管哪一种ingress-controller，实现的机制都大同小异，只是在具体配置上有差异。一般来说，ingress-controller的形式都是一个pod，里面跑着daemon程序和反向代理程序。daemon负责不断监控集群的变化，根据ingress对象生成配置并应用新配置到反向代理，比如nginx-ingress就是动态生成nginx配置，动态更新upstream，并在需要的时候reload程序应用新配置。为了方便，后面的例子都以k8s官方维护的nginx-ingress为例。

### ingress

ingress是一个API对象，和其他对象一样，通过yaml文件来配置。ingress通过http或https暴露集群内部service，给service提供外部URL、负载均衡、SSL/TLS能力以及基于host的方向代理。ingress要依靠ingress-controller来具体实现以上功能。前一小节的图如果用ingress来表示，大概就是如下配置：

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: abc-ingress
  annotations: 
    kubernetes.io/ingress.class: "nginx"
    nginx.ingress.kubernetes.io/use-regex: "true"
spec:
  tls:
  - hosts:
    - api.abc.com
    secretName: abc-tls
  rules:
  - host: api.abc.com
    http:
      paths:
      - backend:
          serviceName: apiserver
          servicePort: 80
  - host: www.abc.com
    http:
      paths:
      - path: /image/*
        backend:
          serviceName: fileserver
          servicePort: 80
  - host: www.abc.com
    http:
      paths:
      - backend:
          serviceName: feserver
          servicePort: 8080
```

与其他k8s对象一样，ingress配置也包含了apiVersion、kind、metadata、spec等关键字段。有几个关注的在spec字段中，tls用于定义https密钥、证书。rule用于指定请求路由规则。这里值得关注的是metadata.annotations字段。在ingress配置中，annotations很重要。前面有说ingress-controller有很多不同的实现，而不同的ingress-controller就可以根据"kubernetes.io/ingress.class:"来判断要使用哪些ingress配置，同时，不同的ingress-controller也有对应的annotations配置，用于自定义一些参数。列如上面配置的'nginx.ingress.kubernetes.io/use-regex: "true"',最终是在生成nginx配置中，会采用location ~来表示正则匹配。

### ingress的部署

ingress的部署，需要考虑两个方面：

1. ingress-controller是作为pod来运行的，以什么方式部署比较好
2. ingress解决了把如何请求路由到集群内部，那它自己怎么暴露给外部比较好

下面列举一些目前常见的部署和暴露方式，具体使用哪种方式还是得根据实际需求来考虑决定。

#### Deployment+LoadBalancer模式的Service

如果要把ingress部署在公有云，那用这种方式比较合适。用Deployment部署ingress-controller，创建一个type为LoadBalancer的service关联这组pod。大部分公有云，都会为LoadBalancer的service自动创建一个负载均衡器，通常还绑定了公网地址。只要把域名解析指向该地址，就实现了集群服务的对外暴露。

#### Deployment+NodePort模式的Service

同样用deployment模式部署ingress-controller，并创建对应的服务，但是type为NodePort。这样，ingress就会暴露在集群节点ip的特定端口上。由于nodeport暴露的端口是随机端口，一般会在前面再搭建一套负载均衡器来转发请求。该方式一般用于宿主机是相对固定的环境ip地址不变的场景。
NodePort方式暴露ingress虽然简单方便，但是NodePort多了一层NAT，在请求量级很大时可能对性能会有一定影响。

#### DaemonSet+HostNetwork+nodeSelector

用DaemonSet结合nodeselector来部署ingress-controller到特定的node上，然后使用HostNetwork直接把该pod与宿主机node的网络打通，直接使用宿主机的80/433端口就能访问服务。这时，ingress-controller所在的node机器就很类似传统架构的边缘节点，比如机房入口的nginx服务器。该方式整个请求链路最简单，性能相对NodePort模式更好。缺点是由于直接利用宿主机节点的网络和端口，一个node只能部署一个ingress-controller pod。比较适合大并发的生产环境使用。

因此，在自建的kubernetes生产环境中，建议使用`DaemonSet+HostNetwork+nodeSelector`方式去部署

### ingress测试

我们来实际部署和简单测试一下ingress。测试集群中已经部署有2个服务gowebhost与gowebip，每次请求能返回容器hostname与ip。测试搭建一个ingress来实现通过域名的不同path来访问这两个服务：

测试ingress使用k8s社区的[ingress-nginx](https://link.segmentfault.com/?url=https%3A%2F%2Fgithub.com%2Fkubernetes%2Fingress-nginx)，部署方式用DaemonSet+HostNetwork。

#### 部署ingress-controller

##### 部署ingress-controller pod及相关资源

官方文档中，部署只要简单的执行一个yaml

```
https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/mandatory.yaml
```

mandatory.yaml这一个yaml中包含了很多资源的创建，包括namespace、ConfigMap、role，ServiceAccount等等所有部署ingress-controller需要的资源，配置太多就不粘出来了，我们重点看下deployment部分：

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-ingress-controller
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app.kubernetes.io/name: ingress-nginx
      app.kubernetes.io/part-of: ingress-nginx
  template:
    metadata:
      labels:
        app.kubernetes.io/name: ingress-nginx
        app.kubernetes.io/part-of: ingress-nginx
      annotations:
        prometheus.io/port: "10254"
        prometheus.io/scrape: "true"
    spec:
      serviceAccountName: nginx-ingress-serviceaccount
      containers:
        - name: nginx-ingress-controller
          image: quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.25.0
          args:
            - /nginx-ingress-controller
            - --configmap=$(POD_NAMESPACE)/nginx-configuration
            - --tcp-services-configmap=$(POD_NAMESPACE)/tcp-services
            - --udp-services-configmap=$(POD_NAMESPACE)/udp-services
            - --publish-service=$(POD_NAMESPACE)/ingress-nginx
            - --annotations-prefix=nginx.ingress.kubernetes.io
          securityContext:
            allowPrivilegeEscalation: true
            capabilities:
              drop:
                - ALL
              add:
                - NET_BIND_SERVICE
            # www-data -> 33
            runAsUser: 33
          env:
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
          ports:
            - name: http
              containerPort: 80
            - name: https
              containerPort: 443
          livenessProbe:
            failureThreshold: 3
            httpGet:
              path: /healthz
              port: 10254
              scheme: HTTP
            initialDelaySeconds: 10
            periodSeconds: 10
            successThreshold: 1
            timeoutSeconds: 10
          readinessProbe:
            failureThreshold: 3
            httpGet:
              path: /healthz
              port: 10254
              scheme: HTTP
            periodSeconds: 10
            successThreshold: 1
            timeoutSeconds: 10
```

可以看到主要使用了“quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.25.0”这个镜像，指定了一些启动参数。同时开放了80与443两个端口，并在10254端口做了健康检查。
我们需要使用daemonset部署到特定node，需要修改部分配置：先给要部署nginx-ingress的node打上特定标签,这里测试部署在"node-1"这个节点。

```
$ kubectl label node node-1 isIngress="true"
```

然后修改上面mandatory.yaml的deployment部分配置为：

```
# 修改api版本及kind
# apiVersion: apps/v1
# kind: Deployment
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  name: nginx-ingress-controller
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
spec:
# 删除Replicas
# replicas: 1
  selector:
    matchLabels:
      app.kubernetes.io/name: ingress-nginx
      app.kubernetes.io/part-of: ingress-nginx
  template:
    metadata:
      labels:
        app.kubernetes.io/name: ingress-nginx
        app.kubernetes.io/part-of: ingress-nginx
      annotations:
        prometheus.io/port: "10254"
        prometheus.io/scrape: "true"
    spec:
      serviceAccountName: nginx-ingress-serviceaccount
      # 选择对应标签的node
      nodeSelector:
        isIngress: "true"
      # 使用hostNetwork暴露服务
      hostNetwork: true
      containers:
        - name: nginx-ingress-controller
          image: quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.25.0
          args:
            - /nginx-ingress-controller
            - --configmap=$(POD_NAMESPACE)/nginx-configuration
            - --tcp-services-configmap=$(POD_NAMESPACE)/tcp-services
            - --udp-services-configmap=$(POD_NAMESPACE)/udp-services
            - --publish-service=$(POD_NAMESPACE)/ingress-nginx
            - --annotations-prefix=nginx.ingress.kubernetes.io
          securityContext:
            allowPrivilegeEscalation: true
            capabilities:
              drop:
                - ALL
              add:
                - NET_BIND_SERVICE
            # www-data -> 33
            runAsUser: 33
          env:
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
          ports:
            - name: http
              containerPort: 80
            - name: https
              containerPort: 443
          livenessProbe:
            failureThreshold: 3
            httpGet:
              path: /healthz
              port: 10254
              scheme: HTTP
            initialDelaySeconds: 10
            periodSeconds: 10
            successThreshold: 1
            timeoutSeconds: 10
          readinessProbe:
            failureThreshold: 3
            httpGet:
              path: /healthz
              port: 10254
              scheme: HTTP
            periodSeconds: 10
            successThreshold: 1
            timeoutSeconds: 10
```

修改完后执行apply,并检查服务

```
$ kubectl apply -f mandatory.yaml

namespace/ingress-nginx created
configmap/nginx-configuration created
configmap/tcp-services created
configmap/udp-services created
serviceaccount/nginx-ingress-serviceaccount created
clusterrole.rbac.authorization.k8s.io/nginx-ingress-clusterrole created
role.rbac.authorization.k8s.io/nginx-ingress-role created
rolebinding.rbac.authorization.k8s.io/nginx-ingress-role-nisa-binding created
clusterrolebinding.rbac.authorization.k8s.io/nginx-ingress-clusterrole-nisa-binding created
daemonset.extensions/nginx-ingress-controller created
# 检查部署情况
$ kubectl get daemonset -n ingress-nginx
NAME                       DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR    AGE
nginx-ingress-controller   1         1         1       1            1           isIngress=true   101s
$ kubectl get po -n ingress-nginx -o wide
NAME                             READY   STATUS    RESTARTS   AGE    IP               NODE     NOMINATED NODE   READINESS GATES
nginx-ingress-controller-fxx68   1/1     Running   0          117s   172.16.201.108   node-1   <none>           <none>
```

可以看到，nginx-controller的pod已经部署在在node-1上了。

##### 暴露nginx-controller

到node-1上看下本地端口：

```
[root@node-1 ~]#  netstat -lntup | grep nginx
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      2654/nginx: master  
tcp        0      0 0.0.0.0:8181            0.0.0.0:*               LISTEN      2654/nginx: master  
tcp        0      0 0.0.0.0:443             0.0.0.0:*               LISTEN      2654/nginx: master  
tcp6       0      0 :::10254                :::*                    LISTEN      2632/nginx-ingress- 
tcp6       0      0 :::80                   :::*                    LISTEN      2654/nginx: master  
tcp6       0      0 :::8181                 :::*                    LISTEN      2654/nginx: master  
tcp6       0      0 :::443                  :::*                    LISTEN      2654/nginx: master
```

由于配置了hostnetwork，nginx已经在node主机本地监听80/443/8181端口。其中8181是nginx-controller默认配置的一个default backend。这样，只要访问node主机有公网IP，就可以直接映射域名来对外网暴露服务了。如果要nginx高可用的话，可以在多个node
上部署，并在前面再搭建一套LVS+keepalive做负载均衡。用hostnetwork的另一个好处是，如果lvs用DR模式的话，是不支持端口映射的，这时候如果用nodeport，暴露非标准的端口，管理起来会很麻烦。

#### 配置ingress资源

部署完ingress-controller，接下来就按照测试的需求来创建ingress资源。

```
# ingresstest.yaml

apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-test
  annotations:
    kubernetes.io/ingress.class: "nginx"
    # 开启use-regex，启用path的正则匹配 
    nginx.ingress.kubernetes.io/use-regex: "true"
spec:
  rules:
    # 定义域名
    - host: test.ingress.com
      http:
        paths:
        # 不同path转发到不同端口
          - path: /ip
            backend:
              serviceName: gowebip-svc
              servicePort: 80
          - path: /host
            backend:
              serviceName: gowebhost-svc
              servicePort: 80
```

部署资源

```
$ kubectl apply -f ingresstest.yaml
```

##### 测试访问

部署好以后，做一条本地host来模拟解析test.ingress.com到node的ip地址。测试访问



#### 关于ingress-nginx

关于ingress-nginx多说几句，上面测试的例子是非常简单的，实际ingress-nginx的有非常多的配置，都可以单独开几篇文章来讨论了。但本文主要想说明ingress，所以不过多涉及。具体可以参考ingress-nginx的[官方文档](https://link.segmentfault.com/?url=https%3A%2F%2Fkubernetes.github.io%2Fingress-nginx%2F)。同时，在生产环境使用ingress-nginx还有很多要考虑的地方，[这篇文章](https://link.segmentfault.com/?url=https%3A%2F%2Fdanielfm.me%2Fposts%2Fpainless-nginx-ingress.html)写得很好，总结了不少最佳实践，值得参考。

#### 最后

- ingress是k8s集群的请求入口，可以理解为对多个service的再次抽象
- 通常说的ingress一般包括ingress资源对象及ingress-controller两部分组成
- ingress-controller有多种实现，社区原生的是ingress-nginx，根据具体需求选择
- ingress自身的暴露有多种方式，需要根据基础环境及业务类型选择合适的方式

#### 参考

[Kubernetes Document](https://link.segmentfault.com/?url=https%3A%2F%2Fkubernetes.io%2Fdocs%2Fconcepts%2Fservices-networking%2Fingress%2F)
[NGINX Ingress Controller Document](https://link.segmentfault.com/?url=https%3A%2F%2Fkubernetes.github.io%2Fingress-nginx%2F)
[Kubernetes Ingress Controller的使用介绍及高可用落地](https://link.segmentfault.com/?url=https%3A%2F%2Fwww.servicemesher.com%2Fblog%2Fkubernetes-ingress-controller-deployment-and-ha%2F)
[通俗理解Kubernetes中Service、Ingress与Ingress Controller的作用与关系](https://link.segmentfault.com/?url=https%3A%2F%2Fimroc.io%2Fposts%2Fkubernetes%2Funderstand-service-ingress-and-ingress-controller%2F)





### 安装ingress-nginx

获取镜像

```shell
[root@k8s-n1 ~]# docker pull  registry.cn-hangzhou.aliyuncs.com/google_containers/nginx-ingress-controller:v0.44.0
[root@k8s-n1 ~]# docker tag  registry.cn-hangzhou.aliyuncs.com/google_containers/nginx-ingress-controller:v0.44.0 k8s.gcr.io/ingress-nginx/controller:v0.44.0
```

下载资源清单文件

```shell
[root@k8s-m1 ~]# wget https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.44.0/deploy/static/provider/baremetal/deploy.yaml

```



### Ingress controller暴露端口

版本：v0.44.0

参考链接：

* https://kubernetes.github.io/ingress-nginx/user-guide/exposing-tcp-udp-services/
* https://www.cnblogs.com/fengjian2016/p/8301668.html
* https://www.cnblogs.com/hongdada/p/11491974.html
* https://segmentfault.com/a/1190000019908991
* https://www.servicemesher.com/blog/kubernetes-ingress-controller-deployment-and-ha/

```shell
https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.44.0/deploy/static/provider/baremetal/deploy.yaml
```

#### 修改deploy文件

修改字段：

1）320行增加了 hostNetwork: true

2）335和336增加了tcp和udp暴露端口的参数

```shell
335             - --tcp-services-configmap=$(POD_NAMESPACE)/tcp-services
336             - --udp-services-configmap=$(POD_NAMESPACE)/udp-services
```

```yaml
318     spec:
319       dnsPolicy: ClusterFirst
320       hostNetwork: true
321       containers:
322         - name: controller
323           image: k8s.gcr.io/ingress-nginx/controller:v0.44.0
324           imagePullPolicy: IfNotPresent
325           lifecycle:
326             preStop:
327               exec:
328                 command:
329                   - /wait-shutdown
330           args:
331             - /nginx-ingress-controller
332             - --election-id=ingress-controller-leader
333             - --ingress-class=nginx
334             - --configmap=$(POD_NAMESPACE)/ingress-nginx-controller
335             - --tcp-services-configmap=$(POD_NAMESPACE)/tcp-services
336             - --udp-services-configmap=$(POD_NAMESPACE)/udp-services
337             - --validating-webhook=:8443
338             - --validating-webhook-certificate=/usr/local/certificates/cert
339             - --validating-webhook-key=/usr/local/certificates/key

```

增加hostNetwork的原因：官方的ingress controller没有绑定到宿主机 80 端口，也就是说前端 Nginx 没有监听宿主机 80 端口，因而需要收到修改资源清单文件，增加一个` hostNetwork: true`



#### 创建Ingress controller

```shell
[root@k8s-m1 k8s]# kubectl apply -f deploy.yaml

# 保证 ingress-nginx-controller正在运行
[root@k8s-m1 k8s]# kubectl get pod --all-namespaces
NAMESPACE       NAME                                        READY   STATUS      RESTARTS   AGE
default         nginx-test1-655ddd64cd-mbcq6                1/1     Running     0          3h15m
default         nginx-test1-655ddd64cd-nlprg                1/1     Running     0          3h15m
default         nginx-test1-655ddd64cd-s5sjf                1/1     Running     0          3h15m
default         nginx-test2-68c89f98c9-2tftz                1/1     Running     0          178m
default         nginx-test2-68c89f98c9-kx6hv                1/1     Running     0          178m
default         nginx-test2-68c89f98c9-sj7zn                1/1     Running     0          178m
default         nginx-test4-6d69f4ff55-dtg2k                1/1     Running     0          170m
default         nginx-test4-6d69f4ff55-j8h7c                1/1     Running     0          170m
default         nginx-test4-6d69f4ff55-wn8px                1/1     Running     0          170m
default         nginx-test5-b4d644bdf-7gxqf                 1/1     Running     0          126m
default         nginx-test5-b4d644bdf-ptshz                 1/1     Running     0          132m
default         nginx-test5-b4d644bdf-q6sj7                 1/1     Running     0          132m
default         nginx-test5-b4d644bdf-zrq66                 1/1     Running     0          132m
ingress-nginx   ingress-nginx-admission-create-g56c4        0/1     Completed   0          15m
ingress-nginx   ingress-nginx-admission-patch-c62nl         0/1     Completed   1          15m
ingress-nginx   ingress-nginx-controller-5fcc5f76dd-crz2g   1/1     Running     0          15m
kube-system     coredns-66bff467f8-fp948                    1/1     Running     3          46h
kube-system     coredns-66bff467f8-vmhwq                    1/1     Running     3          46h
kube-system     etcd-k8s-m1.com                             1/1     Running     4          46h
kube-system     kube-apiserver-k8s-m1.com                   1/1     Running     2          74m
kube-system     kube-controller-manager-k8s-m1.com          1/1     Running     5          46h
kube-system     kube-flannel-ds-8wpjv                       1/1     Running     3          46h
kube-system     kube-flannel-ds-dhwtr                       1/1     Running     4          46h
kube-system     kube-proxy-dd8l9                            1/1     Running     4          46h
kube-system     kube-proxy-wh9bc                            1/1     Running     3          46h
kube-system     kube-scheduler-k8s-m1.com                   1/1     Running     4          46h
kube-system     tiller-deploy-56b574c76d-qbxhj              1/1     Running     1          18h

```



#### 建立暴露tcp端口的configmap

```yaml
[root@k8s-m1 k8s]# cat tcp-services-configmap.yaml 
apiVersion: v1
kind: ConfigMap
metadata:
  name: tcp-services
  namespace: ingress-nginx
data:
  27004: "default/nginx-test5:80"
  7004: "default/nginx-test5:80"

```

```shell
[root@k8s-m1 k8s]# kubectl apply -f tcp-services-configmap.yaml
```

查看ingress-nginx-controller运行的节点

```shell
[root@k8s-m1 k8s]# kubectl get pod --all-namespaces -o wide
...
ingress-nginx   ingress-nginx-controller-5fcc5f76dd-crz2g   1/1     Running     0          16m     192.168.122.101   k8s-n1.com   <none>           <none>

...
```

检测端口运行状态

```shell
[root@k8s-n1 ~]# ss -anpl | grep 7004
tcp    LISTEN     0      511       *:7004                  *:*                   users:(("nginx",pid=24817,fd=41),("nginx",pid=24816,fd=41),("nginx",pid=24815,fd=41),("nginx",pid=24813,fd=41),("nginx",pid=19891,fd=41))
tcp    LISTEN     0      511       *:27004                 *:*                   users:(("nginx",pid=24817,fd=49),("nginx",pid=24816,fd=49),("nginx",pid=24815,fd=49),("nginx",pid=24813,fd=49),("nginx",pid=19891,fd=49))
tcp    LISTEN     0      511    [::]:7004               [::]:*                   users:(("nginx",pid=24817,fd=42),("nginx",pid=24816,fd=42),("nginx",pid=24815,fd=42),("nginx",pid=24813,fd=42),("nginx",pid=19891,fd=42))
tcp    LISTEN     0      511    [::]:27004              [::]:*                   users:(("nginx",pid=24817,fd=50),("nginx",pid=24816,fd=50),("nginx",pid=24815,fd=50),("nginx",pid=24813,fd=50),("nginx",pid=19891,fd=50))
[root@k8s-n1 ~]# 

```

访问测试

```shell
[root@k8s-n1 ~]# curl 192.168.122.101:7004
hello world, this is VERSION 2 for test of nginx
[root@k8s-n1 ~]# curl 192.168.122.101:27004
hello world, this is VERSION 2 for test of nginx

```

