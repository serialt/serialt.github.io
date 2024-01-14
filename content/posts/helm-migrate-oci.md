+++
title = 'Helm Migrate OCI'
date = 2023-09-24T11:05:25+08:00
draft = false
tags = ["helm"]
categories = ["Kubernetes","DevOps"]
+++

## helm oci

### 缘由
Helm 3.8 版本开始，支持使用oci进行存储chart。镜像存储常用的Harbor在v1.6版本开始支持Helm Chart仓库功能，
chart仓库由chartmuseum以插件的方式提供。随着兼容OCI规范的Helm Chart在社区上被更广泛地接受，Helm Chart能以Artifact的形式在Harbor中存储和管理，不再依赖ChartMuseum，因此Harbor在v2.8.0版本中，移除对ChartMuseum的支持。

### registry 仓库使用
基本使用
```shell
### registry
# 登录
helm registry login -u serialt docker.io

# 注销
helm registry logout  docker.io

# pull chart
helm fetch oci://docker.io/serialt/loki-stack --version=2.9.11

# push chart
helm push loki-stack-2.9.11.tgz oci://docker.io/serialt

```
oci支持的其他命令

- `helm pull`
- `helm show`
- `helm template`
- `helm install`
- `helm upgrade`

```shell
$ helm pull oci://localhost:5000/helm-charts/mychart --version 0.1.0
Pulled: localhost:5000/helm-charts/mychart:0.1.0
Digest: sha256:0be7ec9fb7b962b46d81e4bb74fdcdb7089d965d3baca9f85d64948b05b402ff

$ helm show all oci://localhost:5000/helm-charts/mychart --version 0.1.0
apiVersion: v2
appVersion: 1.16.0
description: A Helm chart for Kubernetes
name: mychart
...

$ helm template myrelease oci://localhost:5000/helm-charts/mychart --version 0.1.0
---
# Source: mychart/templates/serviceaccount.yaml
apiVersion: v1
kind: ServiceAccount
...

$ helm install myrelease oci://localhost:5000/helm-charts/mychart --version 0.1.0
NAME: myrelease
LAST DEPLOYED: Wed Oct 27 15:11:40 2021
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
...

$ helm upgrade myrelease oci://localhost:5000/helm-charts/mychart --version 0.2.0
Release "myrelease" has been upgraded. Happy Helming!
NAME: myrelease
LAST DEPLOYED: Wed Oct 27 15:12:05 2021
NAMESPACE: default
STATUS: deployed
REVISION: 2
NOTES:
...
```

### 迁移 chart repo 到 oci 仓库

基于 github action 镜像 helm repo 仓库到 docker hub，以加速 helm repo 到访问。

* [migrate-chart](https://github.com/serialt/migrate-chart)



