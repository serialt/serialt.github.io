

title = 'operator-sdk'
date = 2026-05-26T21:50:01+08:00
draft = false

tags = ["kube-operator","operator-sdk"]
categories = ["Go"]



## 一、Operator SDK 介绍

[Operator SDK](https://github.com/operator-framework/operator-sdk) 是由 Red Hat 开源的用于构建 Kubernetes Operator 的开发框架。它提供了工具、库和标准化模式，帮助开发者高效、一致地创建、测试和打包能够自动化管理 Kubernetes 上复杂应用程序的 Operator。

Operator SDK 基于 Kubernetes controller-runtime 库构建，并通过与 Operator Lifecycle Manager (OLM) 的无缝集成，为 Operator 特定的工具和抽象提供了扩展支持。



#### 为什么需要 Operator SDK

在 Kubernetes 上部署和管理复杂有状态应用时，常常面临生命周期管理复杂、运维门槛高、自动化程度低等挑战。Operator SDK 通过将运维知识编码为软件、提供声明式 API 以及与 kubectl 的一致体验，有效解决了这些问题。

- 将运维专家的知识编码到软件中
- 提供声明式 API 来管理复杂应用
- 支持使用 `kubectl` 操作自定义资源，保持一致的用户体

官方的kube-operator 只支持Go 开发，operator-sdk 可以支持 Go、ansible、helm开发

* **Go-based Operators**: 基于 Go 的 Operator 直接使用 controller-runtime，适合复杂业务逻辑和高度自定义需求。
* **Ansible-based Operators**: Ansible Operator 利用 Ansible playbook 和角色，无需 Go 编程，适合已有自动化脚本的场景。
* **Helm-based Operators**: Helm Operator 基于现有 Helm charts，适合已有 Helm charts 的应用，快速实现 Operator 化。



### 二、开发示例

https://github.com/operator-framework/operator-sdk/releases

#### 1、安装

```bash
🐳 operator-sdk version
operator-sdk version: "v1.42.2", commit: "6001c29067051e1a04e829ea033988b904d1845e", kubernetes version: "1.33.1", go version: "go1.25.7", GOOS: "darwin", GOARCH: "arm64"
```

#### 2、初始化

```bash
mkdir caddy-operator && cd caddy-operator

operator-sdk init --domain example.com --repo github.com/example/caddy-operator
```

##### 创建api

```bash
operator-sdk create api --group web --version v1 --kind Caddy --resource --controller
```

该命令会自动生成自定义资源定义（CRD）、控制器逻辑及相关测试文件

**项目结构说明**

```bash
├── api/
│   └── v1alpha1/          # API 定义
├── config/
│   ├── crd/              # CRD 配置
│   ├── default/          # 默认配置
│   ├── manager/          # Manager 配置
│   ├── rbac/             # RBAC 配置
│   └── samples/          # 示例资源
├── controllers/          # 控制器逻辑
├── Dockerfile           # 容器镜像构建文件
├── Makefile            # 构建和部署命令
├── PROJECT             # 项目元数据
└── main.go             # 主入口文件
```

构建和测试（基于Go开发具体可以参考kube-operator）

```bash
# 生成代码 (kubebuilder 生成器)
make generate

# 生成 manifests (RBAC, CRD 等)
make manifests

# 运行单元测试
make test

# 构建多架构镜像
make docker-buildx IMG=controller:latest

# 本地运行 (开发环境)
make run

# 部署到集群
make deploy IMG=controller:latest
```



### 三、基于 helm

```bash
[root@dev ansible-op]# operator-sdk init --plugins=helm --domain=example.com

[root@dev ansible-op]#  operator-sdk create api --group=web --version=v1 --kind=Caddy

# 需要复制 kustomize 到 bin/kustomize
```



```bash
# 安装crd
[root@dev ansible-op]#  make install

# 构建镜像
[root@dev ansible-op]#  make docker-build docker-push IMG=registry.cn-hangzhou.aliyuncs.com/serialt/caddy-op:v0.0.1 

# 部署
[root@dev ansible-op]#  make deploy IMG=registry.cn-hangzhou.aliyuncs.com/serialt/caddy-op:v0.0.1 


# config/samples/web_v1_caddy.yaml 
apiVersion: web.imau.cc/v1
kind: Caddy
metadata:
  name: caddy
  namespace: dev
spec:
  replicaCount: 5
  image:
    repository: registry.cn-hangzhou.aliyuncs.com/serialt/nginx
    tag: "1.29.6-alpine"
  


[root@dev ansible-op]# kubectl apply -f config/samples/web_v1_caddy.yaml 


# 查看pod
[root@dev samples]# k get pod -n dev
NAME                        READY   STATUS      RESTARTS   AGE
caddy-6cd4f4c7d7-6v9xf      1/1     Running     0          17m
caddy-6cd4f4c7d7-bzslv      1/1     Running     0          17m
caddy-6cd4f4c7d7-pqnsw      1/1     Running     0          17m
caddy-6cd4f4c7d7-t47lg      1/1     Running     0          17m
caddy-6cd4f4c7d7-wgqgq      1/1     Running     0          17m

```

### 四、基于 ansible

```bash
[root@dev ansible-op]#  operator-sdk init --plugins=ansible --domain=imau.cc
[root@dev ansible-op]#  operator-sdk create api --group=webapp --version=v1 --kind=An --generate-role

# 需要复制 kustomize 到 bin/kustomize
```

```yaml

# roles/an/vars/main.yml
image: ""
tag: ""


# roles/an/tasks/main.yml

---
- name: 启动 Nginx 部署
  k8s:
    definition:
      apiVersion: apps/v1
      kind: Deployment
      metadata:
        name: "{{ ansible_operator_meta.name }}-nginx"
        namespace: "{{ ansible_operator_meta.namespace }}"
      spec:
        replicas: 1
        selector:
          matchLabels:
            app: nginx
        template:
          metadata:
            labels:
              app: nginx
          spec:
            containers:
            - name: nginx
              image: "{{ image | default('nginx') }}:{{ tag | default('latest') }}"
              ports:
              - containerPort: 80
              
```

```bash
# 安装crd
[root@dev ansible-op]#  make install

# 构建镜像
[root@dev ansible-op]#  make docker-build docker-push IMG=registry.cn-hangzhou.aliyuncs.com/serialt/ansible-op:v0.0.1

# 部署
[root@dev ansible-op]#  make deploy IMG=registry.cn-hangzhou.aliyuncs.com/serialt/ansible-op:v0.0.1
```

测试文件

```yaml
# config/samples/webapp_v1_an.yaml
apiVersion: webapp.imau.cc/v1
kind: An
metadata:
  labels:
    app.kubernetes.io/name: ansible-op
    app.kubernetes.io/managed-by: kustomize
  name: an-sample
  namespace: dev
spec:
  # TODO(user): Add fields here
  image: registry.cn-hangzhou.aliyuncs.com/serialt/nginx
  tag: 1.29.6-alpine

```

```bash
[root@dev ansible-op]# kubectl apply -f config/samples/webapp_v1_an.yaml 

[root@dev ansible-op]# k get an -A
NAMESPACE   NAME        AGE
dev         an-sample   32m

[root@dev ansible-op]# k get pod -n dev
NAME                               READY   STATUS      RESTARTS   AGE
an-sample-nginx-6bfcf94599-vgj77   1/1     Running     0          18m
```



