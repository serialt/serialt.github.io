title = 'skopeo'
date = 2023-11-07T22:00:42+08:00
draft = false

tags = ["skopeo","image","oci-image-sync"]
categories = ["同步oci镜像"]



# 镜像同步

参考链接：

* https://lework.github.io/2020/04/13/skopeo/
* https://blog.k8s.li/skopeo.html



日常工作中，需要将各种镜像搬到对应的仓库中，docker 适合于构建镜像，将镜像推送于仓库中。镜像被推送到仓库中后，如果需要对镜像进行搬运，在仓库不提供这个功能的情况下，同步镜像是比较困难的。



skopeo 是红帽开源的容器镜像管理工具。相比于docker，它有一下的优点：

* 支持多个平台：skopeo 支持 Linux，Mac 和 Windows。

* 无需 docker 或者 podman：skopeo 可以构建为单一的 cli，不依赖于 docker 服务或者 podman。
* 支持多个 OCI 镜像仓库间同步：支持 OCI 的镜像托管服务，都可以相互同步。
* 支持多架构镜像同步：可以同步多种架构的镜像。
* 镜像验签：skopeo 支持镜像签名，可确保镜像的完整性和可靠性。



## 一、编译skopeo

skopeo 官方并不提供编译好的静态二进制可执行文件，常见的系统源中已经包含了 skopeo，但由于 skopeo 的版本迭代比较快，新的功能也随之增加，部分操作系统里提供的安装包版本可能比较低，无法适用，且 skopeo 大多都是链接了动态库，无法通用于多个 linux 发行版，因此可以借助docker实现skopeo的静态编译。

基于github action构建skopeo: [skopeo](https://github.com/serialt/skopeo)



### 下载 skopeo 源码

```yaml
# download source code
git clone --depth=1  https://github.com/containers/skopeo.git

```

### 构建build镜像的Dockerfile

国内构建则需要修改 alpine 和 go 镜像地址，可直连 github 的可以忽略此步

```dockerfile
FROM golang:1.19-alpine3.16 AS builder

ENV LANG=C.UTF-8
ENV TZ=Asia/Shanghai

ENV CGO_ENABLED=0
ENV GOPROXY=https://goproxy.cn,direct

RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.ustc.edu.cn/g' /etc/apk/repositories
RUN apk update --no-cache && apk add --no-cache ca-certificates 

```

### 构建 build 镜像

````
docker build -t skopeo-build .
````

### 构建 skopeo 静态二进制可执行文件

```shell
cd skopeo/

# 构建 linux amd64 架构
docker run --rm -t -v $PWD:/build skopeo-build sh -c "apk update && apk add gpgme btrfs-progs-dev llvm13-dev gcc musl-dev && cd /build && CGO_ENABLE=0 GO111MODULE=on GOOS=linux GOARCH=amd64 go build -mod=vendor '-buildmode=pie' -ldflags '-extldflags -static' -gcflags '' -tags 'exclude_graphdriver_devicemapper exclude_graphdriver_btrfs containers_image_openpgp' -o ./bin/skopeo-linux-amd64 ./cmd/skopeo "

# 构建 linux arm64 架构
docker run --rm -t -v $PWD:/build skopeo-build sh -c "apk update && apk add gpgme btrfs-progs-dev llvm13-dev gcc musl-dev && cd /build && CGO_ENABLE=0 GO111MODULE=on GOOS=linux GOARCH=arm64 go build -mod=vendor '-buildmode=pie' -ldflags '-extldflags -static' -gcflags '' -tags 'exclude_graphdriver_devicemapper exclude_graphdriver_btrfs containers_image_openpgp' -o ./bin/skopeo-linux-arm64 ./cmd/skopeo  "
```



## 二、skopeo 命令使用

```shell
[root@tc ~]# skopeo -v
skopeo version 1.11.1-dev
[root@tc ~]# skopeo --help
Various operations with container images and container image registries

Usage:
  skopeo [flags]
  skopeo [command]

Available Commands:
  copy                                          Copy an IMAGE-NAME from one location to another
  delete                                        Delete image IMAGE-NAME
  generate-sigstore-key                         Generate a sigstore public/private key pair
  help                                          Help about any command
  inspect                                       Inspect image IMAGE-NAME
  list-tags                                     List tags in the transport/repository specified by the SOURCE-IMAGE
  login                                         Login to a container registry
  logout                                        Logout of a container registry
  manifest-digest                               Compute a manifest digest of a file
  standalone-sign                               Create a signature using local files
  standalone-verify                             Verify a signature using local files
  sync                                          Synchronize one or more images from one location to another

Flags:
      --command-timeout duration   timeout for the command execution
      --debug                      enable debug output
  -h, --help                       help for skopeo
      --insecure-policy            run the tool without any policy check
      --override-arch ARCH         use ARCH instead of the architecture of the machine for choosing images
      --override-os OS             use OS instead of the running OS for choosing images
      --override-variant VARIANT   use VARIANT instead of the running architecture variant for choosing images
      --policy string              Path to a trust policy file
      --registries.d DIR           use registry configuration files in DIR (e.g. for container signature storage)
      --tmpdir string              directory used to store temporary files
  -v, --version                    Version for Skopeo

Use "skopeo [command] --help" for more information about a command.
```



```shell
# 登录与登出 oci
skopeo login -u username  docker.io
skopeo logout docker.io
```

不下载镜像情况下获取镜像信息

```shell
[root@tc ~]# skopeo inspect docker://docker.io/alpine
{
    "Name": "docker.io/library/alpine",
    "Digest": "sha256:eece025e432126ce23f223450a0326fbebde39cdf496a85d8c016293fc851978",
    "RepoTags": [
 
        "20220316",
        "20220328",
        "20220715",
        "20221110",
        "20230208",
        "20230329",
        "20230901",
        "3",
        "3.17",
        "3.17.0",
        "3.17.0_rc1",
        "3.17.1",
        "3.17.2",
        "3.17.3",
        "3.17.4",
        "3.17.5",
        "3.18",
        "3.18.0",
        "3.18.2",
        "3.18.3",
        "3.18.4",
        "edge",
        "latest"
    ],
    "Created": "2023-09-28T21:19:27.801479409Z",
    "DockerVersion": "20.10.23",
    "Labels": null,
    "Architecture": "amd64",
    "Os": "linux",
    "Layers": [
        "sha256:96526aa774ef0126ad0fe9e9a95764c5fc37f409ab9e97021e7b4775d82bf6fa"
    ],
    "LayersData": [
        {
            "MIMEType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
            "Digest": "sha256:96526aa774ef0126ad0fe9e9a95764c5fc37f409ab9e97021e7b4775d82bf6fa",
            "Size": 3401967,
            "Annotations": null
        }
    ],
    "Env": [
        "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
    ]
}
```

>`docker://`: 是使用 Docker Registry HTTP API V2 进行连接远端
>
>`docker.io`: 远程仓库
>
>`alpine`: 镜像名称



获取本地镜像信息

```shell
[root@tc ~]# skopeo inspect docker-daemon:alpine:3
{
    "Name": "docker.io/library/alpine",
    "Digest": "sha256:844bc35fdf7a96e5b6bf5e76e20989a797cc75976fad73275061a36f448b92b9",
    "RepoTags": [],
    "Created": "2023-09-28T21:19:27.801479409Z",
    "DockerVersion": "20.10.23",
    "Labels": null,
    "Architecture": "amd64",
    "Os": "linux",
    "Layers": [
        "sha256:cc2447e1835a40530975ab80bb1f872fbab0f2a0faecf2ab16fbbb89b3589438"
    ],
    "LayersData": [
        {
            "MIMEType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
            "Digest": "sha256:cc2447e1835a40530975ab80bb1f872fbab0f2a0faecf2ab16fbbb89b3589438",
            "Size": 7625728,
            "Annotations": null
        }
    ],
    "Env": [
        "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
    ]
}
```

>`docker-daemon`: docker守护镜像的镜像
>
>`alpine:3`: 本地镜像的名称



### copy镜像

```shell
# skopeo --insecure-policy copy docker://nginx:1.17.6 docker-archive:/tmp/nginx.tar
Getting image source signatures
Copying blob 8ec398bc0356 done
Copying blob 465560073b6f done
Copying blob f473f9fd0a8c done
Copying config f7bb5701a3 done
Writing manifest to image destination
Storing signatures
# ls -alh  /tmp/nginx.tar 
-rw-r--r-- 1 root root 125M 4月  13 15:22 /tmp/nginx.tar
```

>`--insecure-policy`: 用于忽略安全策略配置文件
>
>`docker://nginx:1.17.6`: 该命令将会直接通过 http 下载目标镜像
>
>`docker-archive`: 存储为 /tmp/nginx.tar，此文件可以直接通过 docker load 命令导入



相应的，可以将下载的文件导入到本地

```shell
# skopeo copy docker-archive:/tmp/nginx.tar docker-daemon:nginx:latest
Getting image source signatures
Copying blob 556c5fb0d91b done
Copying blob 49434cc20e95 done
Copying blob 75248c0d5438 done
Copying config f7bb5701a3 done
Writing manifest to image destination
Storing signatures

# docker images nginx
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx               latest              f7bb5701a33c        3 months ago        126MB
COPY

# 也可以将镜像下载到指定目录

# skopeo copy docker://busybox:latest dir:/tmp/busybox
Getting image source signatures
Copying blob 0669b0daf1fb done
Copying config 83aa35aa1c done
Writing manifest to image destination
Storing signatures

# ls -alh /tmp/busybox/
总用量 760K
drwxr-xr-x   2 root root  186 4月  13 15:26 .
drwxrwxrwt. 12 root root 4.0K 4月  13 15:25 ..
-rw-r--r--   1 root root 743K 4月  13 15:26 0669b0daf1fba90642d105f3bc2c94365c5282155a33cc65ac946347a90d90d1
-rw-r--r--   1 root root 1.5K 4月  13 15:26 83aa35aa1c79e4b6957e018da6e322bfca92bf3b4696a211b42502543c242d6f
-rw-r--r--   1 root root  527 4月  13 15:26 manifest.json
-rw-r--r--   1 root root   33 4月  13 15:25 version


#或者从指定目录导入到本地

# skopeo copy dir:/tmp/busybox docker-daemon:busybox:latest
Getting image source signatures
Copying blob 0669b0daf1fb done
Copying config 83aa35aa1c done
Writing manifest to image destination
Storing signatures
# docker images busybox
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
busybox             latest              83aa35aa1c79        4 weeks ago         1.22MB
```



### 删除镜像

```
skopeo delete docker://localhost:5000/nginx:latest
```

### 认证文件

认证文件默认存放在` $HOME/.docker/config.json`

文件内容

```
{
	"auths": {
		"myregistrydomain.com:5000": {
			"auth": "dGVzdHVzZXI6dGVzdHxxxxxxxx",
			"email": "cc@local.com"
		}
	}
}
```

### sync 在 OCI 间同步镜像

使用 docke r在 OCI 间同步镜像的时候，需要先把镜像拉下来，打上 tag ，然后在推送到目的 OCI 上。在这个操作的过程中，即占用了存储，又占用了带宽，在同步大的镜像或者大量的镜像的时候，存储会严重影响镜像在 OCI 间同步的效率。skopeo 正好可以解决这个缺点，skopeo 在同步 OCI 镜像的过程中，只占用带宽，不会把镜像下载到本地。

基于 yaml 文件的同步

```shell
# sync.yaml
ghcr.io:
  images:
    kube-vip/kube-vip:
    - 'v0.6.0'
    - 'v0.4.4'
    k3d-io/k3d-tools:
    - '5.5.2'
```

同步镜像

```shell
skopeo --insecure-policy sync -a --src yaml --dest docker sync.yaml repo.local.com/serialt
```



## 三、同步镜像

目前，常用的 OCI 仓库有：`docker.io`，`quay.io`，`gcr.io`，`registry.k8s.io`，`ghcr.io` 等。众所周知，因为某些原因，这些 OCI 仓库在国内无法访问，而一些项目又严重依赖于存储在这些 OCI 仓库的镜像，虽然有热心的大佬们会把 gcr 和 ghcr 上存储的镜像同步到 docker hub 中，但因为这些被推送到 docker hub 中的镜像不是官方维护的，可能会存在比较大的镜像的同步时间差，某些需要的镜像无法在 docker hub 上找到，同时也容易引起容器镜像的供应链安全问题。因此，可以使用 github action 使用 skopeo 进行同步镜像。

项目地址：[sync-image](https://github.com/serialt/sync-image)



### 1、安装sync-image 和 skopeo

```
wget https://github.com/serialt/skopeo/releases/download/v1.13.3/skopeo-linux-amd64
```

```
go install github.com/serialt/sync-image@latest
```



### 2、配置sync-image yaml 配置文件

```yaml
# config.yaml

# 镜像同步的个数
last: 10
# mcr同步的个数，mcr中包含多个 vscode 容器开发的镜像
mcrLast: 50
autoSyncfile: sync.yaml
# 不同步带有以下关键字的镜像的tag
exclude:
  - 'alpha'
  - 'beta' 
  - 'rc' 
  - 'amd64'
  - 'ppc64le' 
  - 'arm64' 
  - 'arm' 
  - 's390x'   
  - 'SNAPSHOT' 
  - 'snapshot'
  - 'debug' 
  - 'master' 
  - 'latest' 
  - 'main'
  - 'sig'
  - 'sha'
  - 'mips'
# 需要同步的镜像
images:
  docker.elastic.co:
    - elasticsearch/elasticsearch
    - kibana/kibana
    - logstash/logstash
    - beats/filebeat
    - beats/heartbeat
    - beats/packetbeat
    - beats/auditbeat
    - beats/journalbeat
    - beats/metricbeat
    - apm/apm-server
    - app-search/app-search
  quay.io:
    - coreos/flannel
    - ceph/ceph
    - cephcsi/cephcsi
    - csiaddons/k8s-sidecar
    - csiaddons/volumereplication-operator
    - prometheus/prometheus
    - prometheus/alertmanager
    - prometheus/pushgateway
    - prometheus/blackbox-exporter
    - prometheus/node-exporter
    - prometheus-operator/prometheus-config-reloader
    - prometheus-operator/prometheus-operator
    - brancz/kube-rbac-proxy
    - jetstack/cert-manager-webhook
    - jetstack/cert-manager-controller
    - jetstack/cert-manager-cainjector
  k8s.gcr.io:
    - conformance
    - dns/k8s-dns-node-cache
    - metrics-server/metrics-server
    - kube-state-metrics/kube-state-metrics
    - prometheus-adapter/prometheus-adapter
  registry.k8s.io:
    - sig-storage/local-volume-provisioner
    - metrics-server/metrics-server
    - defaultbackend
    - ingress-nginx/controller
    - ingress-nginx/kube-webhook-certgen
    - sig-storage/nfs-subdir-external-provisioner
    - sig-storage/csi-node-driver-registrar
    - sig-storage/csi-provisioner
    - sig-storage/csi-resizer
    - sig-storage/csi-snapshotter
    - sig-storage/snapshot-controller
    - sig-storage/snapshot-validation-webhook
    - sig-storage/nfsplugin
    - sig-storage/csi-attacher
    - sig-storage/livenessprobe
    - defaultbackend-amd64
    - defaultbackend-arm64
    - pause
    - etcd
    - kube-proxy
    - kube-apiserver
    - kube-scheduler
    - kube-controller-manager
    - coredns/coredns
    - build-image/kube-cross
  gcr.io:
    - kaniko-project/executor
  ghcr.io:
    - k3d-io/k3d-tools
    - k3d-io/k3d-proxy
    - kube-vip/kube-vip
  mcr.microsoft.com:
    - devcontainers/base
    - devcontainers/go
  docker.io:
    - flannel/flannel
    - flannel/flannel-cni-plugin
    - calico/kube-controllers
    - serialt/rocky
    - serialt/alma
    - calico/cni
    - calico/pod2daemon-flexvol
    - calico/kube-controllers
    - calico/node
    - rancher/mirrored-flannelcni-flannel-cni-plugin
    - rancher/mirrored-flannelcni-flanne
```





### 3、生成动态同步的 yaml 文件

```
sync-image -c config.yaml
```



### 4、同步镜像

依赖的环境变量

```shell
DEST_HUB_USERNAME
DEST_HUB_PASSWORD
MY_GITHUB_TOKEN
```

同步的shell脚本

```shell

hub="docker.io"
repo="$hub/${DEST_HUB_USERNAME}"

hub2="registry.cn-hangzhou.aliyuncs.com"
repo2="$hub2/${DEST_HUB_USERNAME}"



if [ -f sync.yaml ]; then
   echo "[Start] sync......."
   
    sudo skopeo login -u ${DEST_HUB_USERNAME} -p ${DEST_HUB_PASSWORD} ${hub} \
    && sudo skopeo --insecure-policy sync -a --src yaml --dest docker sync.yaml ${repo} \
    && sudo skopeo --insecure-policy sync -a --src yaml --dest docker custom_sync.yaml ${repo}
    sleep 3
    sudo skopeo login -u ${DEST_HUB_USERNAME} -p ${DEST_HUB_PASSWORD} ${hub2} \
    && sudo skopeo --insecure-policy sync -a --src yaml --dest docker sync.yaml ${repo2} \
    && sudo skopeo --insecure-policy sync -a  --src yaml --dest docker custom_sync.yaml ${repo2}


   echo "[End] done."
   
else
    echo "[Error]not found sync.yaml!"
fi
```





### 5、github action 配置文件

```yaml
name: sync

on:
  push:
    branches:
      - master
      - main
  schedule:
    - cron: "0 2 * * *"
  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - name: Set up Go 
      uses: actions/setup-go@v4
      with:
        go-version: '>=1.21.0'
    - name: Install dependencies
      run: |
        export version=v1.10.0 && export  arch=amd64 && sudo wget https://github.com/lework/skopeo-binary/releases/download/${version}/skopeo-linux-${arch} -O /usr/bin/skopeo && sudo chmod +x /usr/bin/skopeo
        skopeo --version
        go install github.com/serialt/sync-image@latest
    - name: generate_sync_yaml
      env:
        SRC_HUB_USERNAME: ${{ secrets.SRC_HUB_USERNAME }}
        DEST_HUB_USERNAME: ${{ secrets.DEST_HUB_USERNAME }}
        DEST_HUB_PASSWORD: ${{ secrets.DEST_HUB_PASSWORD }}
        MY_GITHUB_TOKEN: ${{ secrets.MY_GITHUB_TOKEN }}
      timeout-minutes: 10
      run: |
        sync-image
    - name: sync image
      env:
        SRC_HUB_USERNAME: ${{ secrets.SRC_HUB_USERNAME }}
        DEST_HUB_USERNAME: ${{ secrets.DEST_HUB_USERNAME }}
        DEST_HUB_PASSWORD: ${{ secrets.DEST_HUB_PASSWORD }}
      run: |
        bash sync.sh
```

