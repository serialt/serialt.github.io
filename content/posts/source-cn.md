+++
title = 'to-mirror'
date = 2023-11-7T20:00:42+08:00
draft = false

tags = ["mirror","source","source-to-cn"]
categories = ["系统或软件换源"]

+++


# 操作系统或者软件换源加速



## 1、操作系统

### rocky

```shell
# 8 base
sed -e 's|^mirrorlist=|#mirrorlist=|g' \
    -e 's|^#baseurl=http://dl.rockylinux.org/$contentdir|baseurl=https://mirrors.ustc.edu.cn/rocky|g' \
    -i.bak   /etc/yum.repos.d/Rocky*.repo 
   
# 8 epel
sed -e 's|^metalink=|#metalink=|g' \
    -e 's|^#baseurl=https\?://download.fedoraproject.org/pub/epel/|baseurl=https://mirrors.ustc.edu.cn/epel/|g' \
    -e 's|^#baseurl=https\?://download.example/pub/epel/|baseurl=https://mirrors.ustc.edu.cn/epel/|g' \
    -i.bak  /etc/yum.repos.d/epel*.repo
    

# 9 base
sed -e 's|^mirrorlist=|#mirrorlist=|g' \
    -e 's|^#baseurl=http://dl.rockylinux.org/$contentdir|baseurl=https://mirrors.ustc.edu.cn/rocky|g' \
    -i.bak   /etc/yum.repos.d/rocky*.repo 
    
# 9 epel
sed -e 's|^metalink=|#metalink=|g' \
    -e 's|^#baseurl=https\?://download.fedoraproject.org/pub/epel/|baseurl=https://mirrors.ustc.edu.cn/epel/|g' \
    -e 's|^#baseurl=https\?://download.example/pub/epel/|baseurl=https://mirrors.ustc.edu.cn/epel/|g' \
    -i.bak /etc/yum.repos.d/epel*.repo
```



### ubuntu

```shell
# http
sed -e "s@http://.*archive.ubuntu.com@http://mirrors.aliyun.com@g" \
    -e "s@http://.*security.ubuntu.com@http://mirrors.aliyun.com@g"  \
    -i.bak  -i /etc/apt/sources.list

# https
sed -e "s@http://.*archive.ubuntu.com@https://mirrors.aliyun.com@g" \
    -e "s@http://.*security.ubuntu.com@https://mirrors.aliyun.com@g"  \
    -i.bak  -i /etc/apt/sources.list
```

### debian

```shell
# debian 12及以上
sed -i 's/\w*.debian.org/mirrors.ustc.edu.cn/g' /etc/apt/sources.list.d/debian.sources 
sed -i "s@http://mirrors.ustc.edu.cn@https://mirrors.ustc.edu.cn@g"  /etc/apt/sources.list.d/debian.sources 

# debian 12以下
sed -i 's/\w*.debian.org/mirrors.ustc.edu.cn/g' /etc/apt/sources.list
sed -i "s@http://mirrors.ustc.edu.cn@https://mirrors.ustc.edu.cn@g" /etc/apt/sources.list

```

### alpine

```shell
# alpine 官方源
https://dl-cdn.alpinelinux.org/alpine/v3.18/main
https://dl-cdn.alpinelinux.org/alpine/v3.18/community

# ustc
sed -i 's/dl-cdn.alpinelinux.org/mirrors.ustc.edu.cn/g' /etc/apk/repositories


# edge 源
echo "https://mirrors.ustc.edu.cn/alpine/edge/main" >> /etc/apk/repositories
echo "https://mirrors.ustc.edu.cn/alpine/edge/community" >> /etc/apk/repositories
echo "https://mirrors.ustc.edu.cn/alpine/edge/testing" >> /etc/apk/repositories
```



## 2、开发语言类

go

```bash
export GOPROXY=https://goproxy.cn,direct

# aliyun
export GOPROXY=https://mirrors.aliyun.com/goproxy/


```



python

```shell
export PIP_MIRROR=mirrors.aliyun.com
echo -e "[global]\nindex-url=https://${PIP_MIRROR}/pypi/simple\n[install]\ntrusted-host=${PIP_MIRROR}" >  /etc/pip.conf 

# 命令配置
pip3 install xxx -i https://mirrors.aliyun.com/pypi/simple/
```

npm

```shell
# 设置全局
npm config set registry https://registry.npmmirror.com

# cmd
npm install -y  --registry=https://registry.npmmirror.com

# 官方地址
https://registry.npmjs.org/

# 阿里云地址
https://registry.npmmirror.com

# 腾讯
http://mirrors.cloud.tencent.com/npm/

# 华为
https://repo.huaweicloud.com/repository/npm/

# 南京大学
https://repo.nju.edu.cn/repository/npm/
```







## 3、容器代理

配置模版

```json
{
    "insecure-registries": [
        "repo.local.com"
    ],
    "exec-opts": [
        "native.cgroupdriver=systemd"
    ],
    "registry-mirrors": [
        "https://docker.mirrors.sjtug.sjtu.edu.cn",
        "https://docker.nju.edu.cn",
        "http://hub-mirror.c.163.com"
    ],
    "log-driver": "json-file",
    "log-opts": {
        "max-size": "100m",
        "max-file": "3"
    },
    "bip":"192.161.20.1/24",
    "dns": [
        "119.29.29.29",
        "223.5.5.5"
    ],
    "data-root": "/var/lib/docker",
    "features": {
        "buildkit": true
    }
}


```

docker registry

```shell
# 上海交大
https://docker.mirrors.sjtug.sjtu.edu.cn

# 南京大学
https://docker.nju.edu.cn

# dockerproxy
https://dockerproxy.com
```

gcr.io

```shell
# 上海交大
https://gcr-io.mirrors.sjtug.sjtu.edu.cn

# 南京大学
https://gcr.nju.edu.cn

# dockerproxy
https://gcr.dockerproxy.com
```

ghcr.io

```shell
# 南京大学
https://htghcr.nju.edu.cn

# dockerproxy
https://ghcr.dockerproxy.com
```

nvcr.io

```shell
# 南京大学
https://nvcr.nju.edu.cn
```

quay.io

```
# 南京大学
https://quay.nju.edu.cn

# dockerproxy
quay.dockerproxy.com
```

registry.k8s.io

```shell
# 南京大学
k8s.mirror.nju.edu.cn

# dockerproxy
k8s.dockerproxy.com
```

Microsoft Artifact Registry

```
mcr.dockerproxy.com
```



