+++
title = 'ktconnect'
date = 2024-01-29T20:26:27+08:00
draft = false

tags = ["ktconnect"]
categories = ["DevOps"]

+++
# KtConnect

官方文档：https://alibaba.github.io/kt-connect/#/zh-cn/guide/quickstart

简单示例：
```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp
spec:
  type: ClusterIP
  selector:
    app: myapp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  labels:
    app: myapp
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: nginx
        image: nginx 
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: 100m
            memory: 100Mi
          limits:
            cpu: 100m
            memory: 100Mi  
 
```

```shell     
sudo ktctl connect -c ~/.kube/config
```

ktctl 会在 default namespace 里启动一个 pod 用于转发流量，这样本地就可以像在集群中一样，访问里面的服务。