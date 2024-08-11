+++

title = 'kubeadm-cert'
date = 2024-08-11T19:32:27+08:00
draft = false

tags = ["kubeadm-cert"]
categories = ["DevOps","k8s"]

+++
## kubeadm-cert

官方的 kubeadm 部署的集群 CA 有效期默认是10年，其他证书有效期默认是1年，1年有效期在真实使用过程中，往往不够，因此需要自行修改 CA 的有效期然后编译；对于已经存在的集群，则可以修改 renew 的时间在让证书有效期达到10年。



### 一、更新证书

#### 1、检查证书有效期

```shell
kubeadm certs check-expiration
```

```shell
[check-expiration] Reading configuration from the cluster...
[check-expiration] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'

CERTIFICATE                EXPIRES                  RESIDUAL TIME   CERTIFICATE AUTHORITY   EXTERNALLY MANAGED
admin.conf                 Aug 06, 2022 19:03 UTC   344d                                    no      
apiserver                  Aug 06, 2022 19:03 UTC   344d            ca                      no      
apiserver-etcd-client      Aug 06, 2022 19:03 UTC   344d            etcd-ca                 no      
apiserver-kubelet-client   Aug 06, 2022 19:03 UTC   344d            ca                      no      
controller-manager.conf    Aug 06, 2022 19:03 UTC   344d                                    no      
etcd-healthcheck-client    Aug 06, 2022 19:03 UTC   344d            etcd-ca                 no      
etcd-peer                  Aug 06, 2022 19:03 UTC   344d            etcd-ca                 no      
etcd-server                Aug 06, 2022 19:03 UTC   344d            etcd-ca                 no      
front-proxy-client         Aug 06, 2022 19:03 UTC   344d            front-proxy-ca          no      
scheduler.conf             Aug 06, 2022 19:03 UTC   344d                                    no      

CERTIFICATE AUTHORITY   EXPIRES                  RESIDUAL TIME   EXTERNALLY MANAGED
ca                      Aug 04, 2031 19:03 UTC   9y              no      
etcd-ca                 Aug 04, 2031 19:03 UTC   9y              no      
front-proxy-ca          Aug 04, 2031 19:03 UTC   9y              no 

```

该命令显示 `/etc/kubernetes/pki` 文件夹中的客户端证书以及 kubeadm（`admin.conf`,`controller-manager.conf` 和 `scheduler.conf`）使用的 KUBECONFIG 文件中嵌入的客户端证书的到期时间 / 剩余时间。

另外，kubeadm 会显示用户证书是否由外部管理（`EXTERNALLY MANAGED`）。在这种情况下，用户应该小心的手动 / 使用其他工具来管理证书更新。

> **说明：**上面的列表中没有包含 kubelet.conf，因为 kubeadm 将 kubelet 配置为[自动更新证书](https://kubernetes.io/docs/tasks/tls/certificate-rotation/)。轮换的证书位于目录 `/var/lib/kubelet/pki`。要修复过期的 kubelet 客户端证书，请参阅 [kubelet 客户端证书轮换失败](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/troubleshooting-kubeadm/#kubelet-client-cert)。



#### 2、更新证书

##### 1）备份集群证书

> **说明：**如果你运行了一个 HA 集群，以下命令需要在所有 Master 节点上执行。

```
cp -a /etc/kubernetes/ /etc/kubernetes-`date +%Y%m%d`
kubectl get cm kubeadm-config -n kube-system -o yaml > /root/kubeadm-config.yaml
```

##### 2）更新证书

执行如下命令更新所有证书

> **说明：**如果你运行了一个 HA 集群，这个命令需要在所有 Master 节点上执行。

```
kubeadm certs renew all
```



出现类似以下输出说明证书更新完成，并且最后一行 `Done renewing certificates. You must restart the kube-apiserver, kube-controller-manager, kube-scheduler and etcd, so that they can use the new certificates.` 提示要求要重启 `kube-apiserver`、`kube-controller-manager`、`kube-scheduler` 和 `etcd` 使其使用新证书。

```
[renew] Reading configuration from the cluster...
[renew] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'

certificate embedded in the kubeconfig file for the admin to use and for kubeadm itself renewed
certificate for serving the Kubernetes API renewed
certificate the apiserver uses to access etcd renewed
certificate for the API server to connect to kubelet renewed
certificate embedded in the kubeconfig file for the controller manager to use renewed
certificate for liveness probes to healthcheck etcd renewed
certificate for etcd nodes to communicate with each other renewed
certificate for serving etcd renewed
certificate for the front proxy client renewed
certificate embedded in the kubeconfig file for the scheduler manager to use renewed

Done renewing certificates. You must restart the kube-apiserver, kube-controller-manager, kube-scheduler and etcd, so that they can use the new certificates.

```

确认证书是否续期成功

```
# kubeadm certs check-expiration
[check-expiration] Reading configuration from the cluster...
[check-expiration] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'

CERTIFICATE                EXPIRES                  RESIDUAL TIME   CERTIFICATE AUTHORITY   EXTERNALLY MANAGED
admin.conf                 Aug 29, 2022 04:20 UTC   364d                                    no      
apiserver                  Aug 29, 2022 04:20 UTC   364d            ca                      no      
apiserver-etcd-client      Aug 29, 2022 04:20 UTC   364d            etcd-ca                 no      
apiserver-kubelet-client   Aug 29, 2022 04:20 UTC   364d            ca                      no      
controller-manager.conf    Aug 29, 2022 04:20 UTC   364d                                    no      
etcd-healthcheck-client    Aug 29, 2022 04:20 UTC   364d            etcd-ca                 no      
etcd-peer                  Aug 29, 2022 04:20 UTC   364d            etcd-ca                 no      
etcd-server                Aug 29, 2022 04:20 UTC   364d            etcd-ca                 no      
front-proxy-client         Aug 29, 2022 04:20 UTC   364d            front-proxy-ca          no      
scheduler.conf             Aug 29, 2022 04:20 UTC   364d                                    no      

CERTIFICATE AUTHORITY   EXPIRES                  RESIDUAL TIME   EXTERNALLY MANAGED
ca                      Aug 04, 2031 19:03 UTC   9y              no      
etcd-ca                 Aug 04, 2031 19:03 UTC   9y              no      
front-proxy-ca          Aug 04, 2031 19:03 UTC   9y              no

```

> **说明：**从命令输出可以看到，所有客户端证书的到期时间均发生了变化，不过不是顺延一年， 而是从执行 renew 成功的时间开始续签一年。



##### 3）修改kube-controller-manager 续签kubelet证书时间

kubelet证书自动更新时间续签是1年，可以修改续签时间或者手动更新kubelet证书

```shell
# 查看帮助手册
kubectl exec -ti kube-controller-manager-master-01 -n kube-system -- kube-controller-manager --help 


```

```shell
# 增加一行     - --cluster-signing-duration=87600h0m0s

# sed命令
cp /etc/kubernetes/manifests/kube-controller-manager.yaml  /root/kube-controller-manager_bak.yaml  

sed -i '/- kube-controller-manager/a \    - --cluster-signing-duration=87600h0m0s' /etc/kubernetes/manifests/kube-controller-manager.yaml  

diff /etc/kubernetes/manifests/kube-controller-manager.yaml   /root/kube-controller-manager_bak.yaml  
14d13
<     - --cluster-signing-duration=87600h0m0s




vi /etc/kubernetes/manifests/kube-controller-manager.yaml
-------
  name: kube-controller-manager
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-controller-manager
    - --cluster-signing-duration=87600h0m0s
    - --allocate-node-cidrs=true
    - --authentication-kubeconfig=/etc/kubernetes/controller-manager.conf
    - --authorization-kubeconfig=/etc/kubernetes/controller-manager.conf
    - --bind-address=127.0.0.1
    - --client-ca-file=/etc/kubernetes/pki/ca.crt
--------
```





##### 4）启用证书

依次在所有 Master 节点上执行如下命令重启 `kube-apiserver`、`kube-controller-manager`、`kube-scheduler` 和 `etcd` 使其启用新证书。

```
# kubectl delete pod -n kube-system xxxx_pod_name

kubectl delete pod -n kube-system kube-apiserver-cnmesmn1
kubectl delete pod -n kube-system kube-controller-manager-cnmesmn1
kubectl delete pod -n kube-system kube-scheduler-cnmesmn1
kubectl delete pod -n kube-system etcd-cnmesmn1


```

```
docker ps |grep -E 'k8s_kube-apiserver|k8s_kube-controller-manager|k8s_kube-scheduler|k8s_etcd' | awk -F ' ' '{print $1}' |xargs docker restart

```



> 警告：执行完一台后请务必通过 `kubectl get pod -n kube-system` 确认对应 Master 节点上的 `kube-apiserver`、`kube-controller-manager`、`kube-scheduler` 和 `etcd` 服务 Pod 均处于 `Running` 状态并就绪后再开始操作下一台

##### 5）替换 config 文件

执行如下命令替换 config 文件

> **说明：**如果你运行了一个 HA 集群，这个命令需要在所有 Master 节点上执行

```
cp -a -i /etc/kubernetes/admin.conf /root/.kube/config

```

##### 6）重启所有node节点 kubelet

```
systemctl restart kubelet
```



##### 7）检查kubelet的证书更新

```
kubectl get csr
```

```
 openssl x509 -in /var/lib/kubelet/pki/kubelet-client-current.pem -noout -text
```

修改系统时间测试

```
 
date -s "03/23/2024 18:15:55"
```



#### 3、手动更新kubelet证书 [可选]

https://kubernetes.io/zh-cn/docs/tasks/administer-cluster/kubeadm/kubeadm-certs/

1）删除kubelet配置

```
mkdir -p ./kubelet_bak/var_lib
cp /etc/kubernetes/kubelet.conf ./kubelet_bak/
cp /var/lib/kubelet/pki/kubelet-client* ./kubelet_bak/var_lib/
```

2）在master节点生成kubelet证书

```
# $NODE 必须设置为集群中需要更新的节点名字
export NODE=cnmesmn1
kubeadm kubeconfig user --org system:nodes --client-name system:node:${NODE} --config kubeadm-init.yml > ${NODE}-kubelet.conf

cat ${NODE}-kubelet.conf >   /etc/kubernetes/kubelet.conf 
```

```
export NODE=node-01
kubeadm kubeconfig user --org system:nodes --client-name system:node:${NODE} --config kubeadm-init.yml > ${NODE}-kubelet.conf
```



3）将生成的节点证书替换对应的node节点上的证书，作为 `/etc/kubernetes/kubelet.conf`



4）重启kubelet，等待 `/var/lib/kubelet/pki/kubelet-client-current.pem` 重新创建

```
rm -rf /var/lib/kubelet/pki/kubelet-client* 
systemctl restart kubelet
```

5）手动编辑 `kubelet.conf` 指向轮换的 kubelet 客户端证书，方法是将 `client-certificate-data` 和 `client-key-data` 替换为：

```shell
sed -i "/client-certificate/a \    client-certificate: /var/lib/kubelet/pki/kubelet-client-current.pem"  /etc/kubernetes/kubelet.conf
sed -i '/client-key/a \    client-key: /var/lib/kubelet/pki/kubelet-client-current.pem'  /etc/kubernetes/kubelet.conf
sed -i '/client-certificate-data/d'  /etc/kubernetes/kubelet.conf
sed -i '/client-key-data/d'  /etc/kubernetes/kubelet.conf



```

6）再次重启kubelet，确保节点状况变为 `Ready`

```
systemctl restart kubelet
```

检查证书有效期：

```
kubeadm certs check-expiration

for item in `find /etc/kubernetes/pki -maxdepth 2 -name "*.crt"`;do echo "==========$item=========";openssl x509 -in $item -text -noout| grep Not;done
```





### 二、编译kubeadm

kubeadm 默认证书为一年，一年过期后，会导致 api service 不可用，使用过程中会出现：x509: certificate has expired or is not yet valid，Google 建议通过不停更新版本来自动更新证书，若需要长期有效，建议自己编译kubeadm

参考文档：https://zhuanlan.zhihu.com/p/444057661

以下兼容版本：

- 1.17.0
- 1.18.0
- 1.19.0
- 1.20.0
- 1.21.0
- 1.22.0
- 1.23.0

#### 1、下载源码

```shell
[sugar@imau github]$ wget https://github.com/kubernetes/kubernetes/archive/v1.21.14.tar.gz

[sugar@imau github]$ tar -xf v1.21.14.tar.gz 
[sugar@imau kubernetes-1.21.14]$ 
```

#### 2、修改ca有效期

```go
# ca证书默认10年
[sugar@imau kubernetes-1.21.14]$ vim staging/src/k8s.io/client-go/util/cert/cert.go
// 这个方法里面 NotAfter:              now.Add(duration365d * 10).UTC()
// 默认有效期就是 10 年，改成 100 年
// 输入 /NotAfter 查找，回车定位
func NewSelfSignedCACert(cfg Config, key crypto.Signer) (*x509.Certificate, error) {
        now := time.Now()
        tmpl := x509.Certificate{
                SerialNumber: new(big.Int).SetInt64(0),
                Subject: pkix.Name{
                        CommonName:   cfg.CommonName,
                        Organization: cfg.Organization,
                },
                NotBefore:             now.UTC(),
                // NotAfter:              now.Add(duration365d * 10).UTC(),
                NotAfter:              now.Add(duration365d * 100).UTC(),
                KeyUsage:              x509.KeyUsageKeyEncipherment | x509.KeyUsageDigitalSignature | x509.KeyUsageCertSign,
                BasicConstraintsValid: true,
                IsCA:                  true,
        }

        certDERBytes, err := x509.CreateCertificate(cryptorand.Reader, &tmpl, &tmpl, key.Public(), key)
        if err != nil {
                return nil, err
        }
        return x509.ParseCertificate(certDERBytes)
}
```

修改证书有效期为 100 年（默认为 1 年）

```go
vim ./cmd/kubeadm/app/constants/constants.go
// 就是这个常量定义 CertificateValidity，改成 * 100 年
// 输入 /CertificateValidity 查找，回车定位
const (
        // KubernetesDir is the directory Kubernetes owns for storing various configuration files
        KubernetesDir = "/etc/kubernetes"
        // ManifestsSubDirName defines directory name to store manifests
        ManifestsSubDirName = "manifests"
        // TempDirForKubeadm defines temporary directory for kubeadm
        // should be joined with KubernetesDir.
        TempDirForKubeadm = "tmp"

        // CertificateValidity defines the validity for all the signed certificates generated by kubeadm
        // CertificateValidity = time.Hour * 24 * 365
        CertificateValidity = time.Hour * 24 * 365 * 100

        // CACertAndKeyBaseName defines certificate authority base name
        CACertAndKeyBaseName = "ca"
        // CACertName defines certificate name
        CACertName = "ca.crt"
        // CAKeyName defines certificate name
        CAKeyName = "ca.key"
```

验证一下已经正确修改：

```
cat ./staging/src/k8s.io/client-go/util/cert/cert.go | grep NotAfter
cat ./cmd/kubeadm/app/constants/constants.go | grep CertificateValidity

git diff
```

#### 3、编译

为了不影响机器环境，采用docker 中编译

1）制作build的基础镜像

```dockerfile
FROM rockylinux:9

LABEL mantainer="serialt <tserialt@gmail.com> rocky9 image"

# change yum repo to ustc
RUN   sed -e 's|^mirrorlist=|#mirrorlist=|g' \
    -e 's|^#baseurl=http://dl.rockylinux.org/$contentdir|baseurl=https://mirrors.ustc.edu.cn/rocky|g' \
    -i.bak \
    /etc/yum.repos.d/rocky*.repo && \
    yum -y install epel-release && \
    sed -e 's|^metalink=|#metalink=|g' \
    -e 's|^#baseurl=https\?://download.fedoraproject.org/pub/epel/|baseurl=https://mirrors.ustc.edu.cn/epel/|g' \
    -e 's|^#baseurl=https\?://download.example/pub/epel/|baseurl=https://mirrors.ustc.edu.cn/epel/|g' \
    -i.bak \
    /etc/yum.repos.d/epel*.repo

# install base software
RUN  yum -y install git vim-enhanced bash-completion wget gcc make golang && \
    yum groupinstall "Development Tools" -y && \
    yum -y upgrade  && \
    yum clean all



ENV LANG zh_CN.UTF-8
ENV TZ "Asia/Shanghai"



WORKDIR /root

EXPOSE 22 80 443

CMD /bin/bash

```

```shell
# 编译kubeadm, 这里主要编译 kubeadm 即可
make all WHAT=cmd/kubeadm GOFLAGS=-v

# 编译 kubelet
# make all WHAT=cmd/kubelet GOFLAGS=-v

# 编译 kubectl
# make all WHAT=cmd/kubectl GOFLAGS=-v
```

CentOS：

```shell
yum groupinstall "Development Tools" -y #gcc, make etc.
yum install rsync jq -y
```

Ubuntu：

```shell
sudo apt install build-essential #(Following command will install essential commands like gcc, make etc.)
sudo apt install rsync jq -y
```

GoLang 环境

查看 kube-cross 的 TAG 版本号

```
[root@51 kubernetes-1.21.14]# cat ./build/build-image/cross/VERSION
v1.21.0-go1.16.15-buster.0
```

* GitHub：https://github.com/serialt/kubeadm

  

#### 4、更新证书

如果是使用原版 kubeadm 安装之后，可以手动执行命令更新证书有效期到 100 年，但因为ca有效期是10年，使用kubeadm更新证书的时候并不会更新ca证书，建议修改到9年；或者使用编译的kubeadm 去创建集群

可以先备份证书，证书在 /etc/kubernetes/pki

1）检查证书到期时间

```
kubeadm certs check-expiration
# 早期版本 (1.19 及之前版本) 命令如下
#kubeadm alpha certs check-expiration
```

```shell
[root@master-03 ~]# kubeadm certs check-expiration
[check-expiration] Reading configuration from the cluster...
[check-expiration] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'

CERTIFICATE                EXPIRES                  RESIDUAL TIME   CERTIFICATE AUTHORITY   EXTERNALLY MANAGED
admin.conf                 Nov 13, 2023 14:19 UTC   337d            ca                      no      
apiserver                  Nov 13, 2023 14:19 UTC   337d            ca                      no      
apiserver-etcd-client      Nov 13, 2023 14:19 UTC   337d            etcd-ca                 no      
apiserver-kubelet-client   Nov 13, 2023 14:19 UTC   337d            ca                      no      
controller-manager.conf    Nov 13, 2023 14:19 UTC   337d            ca                      no      
etcd-healthcheck-client    Nov 13, 2023 14:19 UTC   337d            etcd-ca                 no      
etcd-peer                  Nov 13, 2023 14:19 UTC   337d            etcd-ca                 no      
etcd-server                Nov 13, 2023 14:19 UTC   337d            etcd-ca                 no      
front-proxy-client         Nov 13, 2023 14:19 UTC   337d            front-proxy-ca          no      
scheduler.conf             Nov 13, 2023 14:19 UTC   337d            ca                      no      

CERTIFICATE AUTHORITY   EXPIRES                  RESIDUAL TIME   EXTERNALLY MANAGED
ca                      Nov 10, 2032 14:19 UTC   9y              no      
etcd-ca                 Nov 10, 2032 14:19 UTC   9y              no      
front-proxy-ca          Nov 10, 2032 14:19 UTC   9y              no      
[root@master-03 ~]# 



```



2）续订全部证书

```shell
[root@master-03 opt]# ./kubeadm-v1.21.14-linux-amd64-100y certs renew all
[renew] Reading configuration from the cluster...
[renew] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'

certificate embedded in the kubeconfig file for the admin to use and for kubeadm itself renewed
certificate for serving the Kubernetes API renewed
certificate the apiserver uses to access etcd renewed
certificate for the API server to connect to kubelet renewed
certificate embedded in the kubeconfig file for the controller manager to use renewed
certificate for liveness probes to healthcheck etcd renewed
certificate for etcd nodes to communicate with each other renewed
certificate for serving etcd renewed
certificate for the front proxy client renewed
certificate embedded in the kubeconfig file for the scheduler manager to use renewed

Done renewing certificates. You must restart the kube-apiserver, kube-controller-manager, kube-scheduler and etcd, so that they can use the new certificates.
[root@master-03 opt]# systemctl restart kubelet
[root@master-03 opt]# 
```

重启kubelet

```
[root@master-03 opt]# systemctl restart kubelet
```

```shell
[root@master-03 opt]# kubeadm certs check-expiration
[check-expiration] Reading configuration from the cluster...
[check-expiration] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'

CERTIFICATE                EXPIRES                  RESIDUAL TIME   CERTIFICATE AUTHORITY   EXTERNALLY MANAGED
admin.conf                 Nov 16, 2122 17:06 UTC   99y             ca                      no      
apiserver                  Nov 16, 2122 17:06 UTC   99y             ca                      no      
apiserver-etcd-client      Nov 16, 2122 17:06 UTC   99y             etcd-ca                 no      
apiserver-kubelet-client   Nov 16, 2122 17:06 UTC   99y             ca                      no      
controller-manager.conf    Nov 16, 2122 17:06 UTC   99y             ca                      no      
etcd-healthcheck-client    Nov 16, 2122 17:06 UTC   99y             etcd-ca                 no      
etcd-peer                  Nov 16, 2122 17:06 UTC   99y             etcd-ca                 no      
etcd-server                Nov 16, 2122 17:06 UTC   99y             etcd-ca                 no      
front-proxy-client         Nov 16, 2122 17:06 UTC   99y             front-proxy-ca          no      
scheduler.conf             Nov 16, 2122 17:06 UTC   99y             ca                      no      

CERTIFICATE AUTHORITY   EXPIRES                  RESIDUAL TIME   EXTERNALLY MANAGED
ca                      Nov 10, 2032 14:19 UTC   9y              no      
etcd-ca                 Nov 10, 2032 14:19 UTC   9y              no      
front-proxy-ca          Nov 10, 2032 14:19 UTC   9y              no      
[root@master-03 opt]# 
```

