+++
title = 'multipass'
date = 2023-11-11T10:28:11+08:00
draft = false

tags = ["multipass","vm"]
categories = ["虚拟化"]

+++

​		Multipass 是一个轻量虚拟机管理器，是由 Ubuntu 运营公司 Canonical 所推出的开源项目。运行环境支持 Linux、Windows、macOS。在不同的操作系统上，使用的是不同的虚拟化技术。在 Linux 上使用的是 KVM、Window 上使用 Hyper-V、macOS 中使用 HyperKit 以最小开销运行VM，支持在笔记本模拟小型云。Multipass唯一的遗憾是支持Linux版本只有Ubuntu。

同时，Multipass 提供了一个命令行界面来启动和管理 Linux 实例。下载一个全新的镜像需要几秒钟的时间，并且在几分钟内就可以启动并运行 VM。



github地址：https://github.com/canonical/multipass

官网：https://multipass.run/



### 安装

`Mac`：

```
# 1）brew 安装
brew install --cask multipass

# 2)github或者官网下二进制安装
```

`Linux`：

ubuntu官方只release了snap安装版本

```
sudo snap install multipass
```

`Windows`：

github下载安装包



### 使用

1）查找可以下载的ubuntu镜像（multipass官方镜像只有ubuntu）

```shell
[root@sugar ~]$ multipass find
Image                       Aliases           Version          Description
18.04                       bionic            20221014         Ubuntu 18.04 LTS
20.04                       focal             20221018         Ubuntu 20.04 LTS
22.04                       jammy,lts         20221101.1       Ubuntu 22.04 LTS
anbox-cloud-appliance                         latest           Anbox Cloud Appliance
charm-dev                                     latest           A development and testing environment for charmers
docker                                        latest           A Docker environment with Portainer and related tools
jellyfin                                      latest           Jellyfin is a Free Software Media System that puts you in control of managing and streaming your media.
minikube                                      latest           minikube is local Kubernetes
```



2）创建虚拟机

参数：

```
-n, --name: 名称
-c, --cpus: cpu核心数, 默认: 1
-m, --mem: 内存大小, 默认: 1G
-d, --disk: 硬盘大小, 默认: 5G
```

创建虚拟机时需要联网下载镜像

```shell
# 使用最新版LTS镜像
[root@sugar ~]$ multipass launch -n vm01 -c 1 -m 1G -d 10G

# 设置使用的镜像
[root@sugar ~]$ multipass launch focal -n vm01 -c 1 -m 1G -d 10G

# 设置网络
# name 网卡名
# mode dhcp方式，auto或者manual，默认auto
# mac 设置mac地址
[root@sugar ~]$ multipass launch --network en0 --network name=bridge0,mode=manual
```

修改配置

```
[root@sugar ~]$ multipass set local.<instance-name>.(cpus|disk|memory) xxx
```



3）与虚拟机交互

```shell
# 默认是以ubuntu用户进入
[root@sugar ~]$ multipass shell vm01

# 在虚拟机外执行命令
[root@sugar ~]$ multipass exec vm01 -- pwd
```



4）启动与关闭

```shell
[root@sugar ~]$ multipass start vm01

[root@sugar ~]$ multipass start vm01 vm02
[root@sugar ~]$ multipass start --all

[root@sugar ~]$ multipass suspend vm01
[root@sugar ~]$ multipass suspend --all

[root@sugar ~]$ multipass stop vm01
[root@sugar ~]$ multipass stop --all

[root@sugar ~]$ multipass delete vm01
[root@sugar ~]$ multipass delete --all

# 从一个删除的实例中恢复
[root@sugar ~]$ multipass recover keen-yak


# 移除一个实例
[root@sugar ~]$ multipass delete keen-yak
[root@sugar ~]$ multipass purge
[root@sugar ~]$ multipass list
No instances found.

# 或者
[root@sugar ~]$$ multipass delete --purge keen-yak
```



5）数据共享

mount

```shell
$ multipass mount $HOME keen-yak
$ multipass info keen-yak
…
Mounts:         /home/michal => /home/michal

# 挂载指定路径
$ multipass mount $HOME keen-yak:/some/path

# 查看信息
$ multipass info keen-yak         

# 卸载
$ multipass umount keen-yak

```

传输文件

```shell
$ multipass transfer keen-yak:/etc/crontab keen-yak:/etc/fstab /home/michal
$ ls -l /home/michal/crontab /home/michal/fstab
-rw-r--r-- 1 michal michal 722 Oct 18 12:13 /home/michal/crontab
-rw-r--r-- 1 michal michal  82 Oct 18 12:13 /home/michal/fstab
$ multipass transfer /home/michal/crontab /home/michal/fstab keen-yak:
$ multipass exec keen-yak -- ls -l crontab fstab
-rw-rw-r-- 1 ubuntu ubuntu 722 Oct 18 12:14 crontab
-rw-rw-r-- 1 ubuntu ubuntu  82 Oct 18 12:14 fstab
```



6）容器自动化

为了保持开发环境和线上环境一致性 同时节省部署时间 **multipass** 给我们提供了 **--cloud-init** 选项进行容器启动初始化配置:

```
multipass launch --name ubuntu --cloud-init config.yaml

```

上面 **config.yaml** 则是容器的初始化配置文件，例如，我们想在初始化容器的时候，自动下载安装 **Node.js**，内容如下：



```
#cloud-config
runcmd:
  - curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
  - sudo apt-get install -y nodejs
```

`runcmd` 可以指定容器 **首次启动** 时运行的命令

```
凡是用户自定义的cloud-init的配置文件,必须以#cloud-config开头，这是cloud-init识别它的方式。
```

**yaml** 配置文件参考链接：https://cloudinit.readthedocs.io/en/latest/topics/examples.html?highlight=lock-passwd#including-users-and-groups

