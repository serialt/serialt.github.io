+++
title = 'OpenConnect'
date = 2024-03-23T10:37:07+08:00
draft = false

tags = ["OpenConnect","vpn"]
categories = ["DevOps"]

+++
## OpenConnect

参考文档

* https://github.com/huataihuang/cloud-atlas-draft/blob/master/security/vpn/openconnect/deploy_ocserv_vpn_server_on_ubuntu.md
* https://www.cnblogs.com/grafin/p/17785264.html
* https://cn.linux-console.net/?p=22087



OpenConnect起源于对Cisco AnyConnect VPN客户端的开源替代方案的需求。Cisco AnyConnect是一款商业VPN解决方案，用于建立安全的远程连接。然而，由于其闭源和商业性质，社区对于一个开源的替代品的需求逐渐增加。OpenConnect的发展旨在填补这一空白，为用户提供一个自由、灵活且安全的VPN连接选项。

**工作原理**

OpenConnect的工作原理基于与Cisco AnyConnect服务器建立的安全连接。它使用SSL和DTLS协议进行身份验证和加密通信。用户首先提供所需的连接参数，例如服务器地址、用户名和密码。然后，OpenConnect使用这些参数与服务器进行握手和认证。一旦身份验证成功，VPN连接将建立起来，用户可以通过加密通道安全地传输数据。

软件分为客户端和服务端：

* 服务端：ocserv

* 客户端：openconnect

客户端支持的vpn服务有

* Cisco AnyConnect SSL VPN
* Juniper Network Connect
* GlobalProtect SSL VPN
* F5 BIG-IP SSL VPN
* FortiGate SSL VPN
* Array Networks SSL VPN

```shell
Set VPN protocol:
      --protocol=anyconnect       Compatible with Cisco AnyConnect SSL VPN, as well as ocserv (default)
      --protocol=nc               Compatible with Juniper Network Connect
      --protocol=gp               Compatible with Palo Alto Networks (PAN) GlobalProtect SSL VPN
      --protocol=pulse            Compatible with Pulse Connect Secure SSL VPN
      --protocol=f5               Compatible with F5 BIG-IP SSL VPN
      --protocol=fortinet         Compatible with FortiGate SSL VPN
      --protocol=array            Compatible with Array Networks SSL VPN
```



### 一、部署

#### 1、安装 ocserv

`RHEL 系列`

ocserv 已经在 epel 仓库中提供了，所以可以直接通过 yum 安装

```shell
[root@dev ~]# yum install epel-release

# gnutls-utils 包含证书的制作工具 certtool
[root@dev ~]# yum install ocserv gnutls-utils
```

#### 2、生成证书

```shell
[root@dev ~]# mkdir /opt/anyconnect
[root@dev ~]# cd /opt/anyconnect/

# 生成 CA 证书
[root@dev ~]# certtool --generate-privkey --outfile ca-key.pem
cat >ca.tmpl <<EOF
cn = "VPN CA"
organization = "Local Corp"
serial = 1
expiration_days = 36500
ca
signing_key
cert_signing_key
crl_signing_key
EOF

[root@dev ~]# certtool --generate-self-signed --load-privkey ca-key.pem \
--template ca.tmpl --outfile ca-cert.pem

# 复制 CA 证书到 ocserv 配置目录中
[root@dev ~]# cp ca-key.pem /etc/ocserv/

# 创建服务端证书
[root@dev ~]# certtool --generate-privkey --outfile server-key.pem
[root@dev ~]# cat >server.tmpl <<EOF
cn = "VPN server"
organization = "Local"
serial = 2
expiration_days = 3650
encryption_key
signing_key
tls_www_server
EOF

[root@dev ~]# certtool --generate-certificate --load-privkey server-key.pem \
--load-ca-certificate ca-cert.pem --load-ca-privkey ca-key.pem \
--template server.tmpl --outfile server-cert.pem

[root@dev ~]# cp server-cert.pem server-key.pem /etc/ocserv/
```

#### 3、配置ocserv

`/etc/ocserv/ocserv.conf     `

```toml
#ocserv支持多种认证方式，这是自带的密码认证，使用ocpasswd创建密码文件
#ocserv还支持证书认证，可以通过Pluggable Authentication Modules (PAM)使用radius等认证方式
auth = "plain[passwd=/etc/ocserv/ocpasswd]"
#指定替代的登录方式，这里使用证书登录作为第二种登录方式

#指定替代的登录方式，这里使用证书登录作为第二种登录方式
#enable-auth = "certificate"

#tcp和udp端口
tcp-port = 44333
udp-port = 44333


#运行用户和组
run-as-user = ocserv
run-as-group = ocserv

# socket文件
socket-file = /var/run/ocserv.sock

#证书路径
server-cert = /etc/ocserv/server-cert.pem
server-key = /etc/ocserv/server-key.pem
#ca路径
ca-cert = /etc/ocserv/ca-cert.pem

# 开启lz4压缩
compression = true

# 隔离工作，默认不动
isolate-workers = true

# 最大客户端数量，0表示无限数量
max-clients = 16

# 同一用户可以同时登陆的客户端数量
max-same-clients = 5

# 默认不动
rate-limit-ms = 100
# 服务器统计重置时间，不动
server-stats-reset-time = 604800


# 保持连接，每隔多少秒向客户端发送连接数据包，防止断线。
# IOS系统5分钟会关闭后台数据通讯，然后就会断线。
# 因此将keepalive和mobile-dpd设置成200秒
keepalive = 200
dpd = 90
mobile-dpd = 200

# udp端口无传输25秒后转成tcp端口
switch-to-tcp-timeout = 25

# 启用MTU转发以优化性能
try-mtu-discovery = true

# 空闲断开时间，如果想无限期连接，注释这两行
# idle-timeout=1200
# mobile-idle-timeout=2400

# 仅使用TLS1.2以上版本
cert-user-oid = 0.9.2342.19200300.100.1.1
tls-priorities = "NORMAL:%SERVER_PRECEDENCE:%COMPAT:-RSA:-VERS-SSL3.0:-ARCFOUR-128:-VERS-TLS1.0:-VERS-TLS1.1"

# 认证超时时间
auth-timeout = 240
# 最小重新认证时间
min-reauth-time = 300
max-ban-score = 80
ban-reset-time = 1200
cookie-timeout = 324000
deny-roaming = false
rekey-time = 172800
rekey-method = ssl
use-occtl = true
pid-file = /var/run/ocserv.pid
device = vpns
predictable-ips = true

# 默认域名，修改为你的域名或ip地址，这个设置没有任何作用
default-domain = xxx.com:4433

# 配置自定义私有IP地址范围(vpn客户端)，注释默认的两行
#ipv4-network = 192.168.1.0
#ipv4-netmask = 255.255.255.0
ipv4-network = 10.70.25.0
ipv4-netmask = 255.255.255.0

# 以VPN隧道传输所有DNS查询
tunnel-all-dns = true

# 更改DNS服务器(国内服务器就填写国内dns)
dns = 8.8.8.8
dns = 1.1.1.1

# 允许思科客户端连接
cisco-client-compat = true

# 以下路由表不通过VPN隧道，直接本地网络连接
# 一定要添加自己服务器的ip地址，否则连上VPN后打不开自己的网站
no-route = x.x.x.x/255.255.255.255


#只允许访问下面的网段(推送的路由)
route = 192.168.0.0/255.255.0.0
route = 172.16.0.0/255.255.0.0


```



#### 4、创建用户

```shell
#username为你要添加的用户名
[root@dev ~]# ocpasswd -c /etc/ocserv/ocpasswd username


# 检查内核转发
[root@dev ~]# cat /proc/sys/net/ipv4/ip_forward

# 开启内核转发
[root@dev ~]# echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
[root@dev ~]# sysctl -p
```



```shell
# 配置iptables规则(不需要配置)
# 对指定的表 table 进行操作，添加一个规则，把192.168.111.0的流量指定到ens36出去
iptables -t nat -A POSTROUTING -s 192.168.111.0 -o ens36 -j MASQUERADE

iptables -A FORWARD -i vpns+ -j ACCEPT
iptables -A FORWARD -o vpns+ -j ACCEPT

# 保存路由表
iptables-save > /etc/sysconfig/iptables
```

```shell
# 放行端口(firewalld配置)
firewall-cmd --permanent --add-port=44333/tcp
firewall-cmd --permanent --add-port=44333/udp
firewall-cmd --add-masquerade --permanent #必须配置，重要
firewall-cmd --reload
```

#### 5、测试

测试服务启动

```sh
[root@dev ~]# ocserv -c /etc/ocserv/ocserv.conf -f -d 10
```

启动服务

```sh
[root@dev ~]# systemctl enable ocserv
[root@dev ~]# systemctl start ocserv
```



#### 6 客户端连接

```shell
# RHEL
[root@dev ~]# yum install epel-release
[root@dev ~]# yum -y install openconnect
```

```shell
# 密码连接
[root@dev ~]# echo -n password | openconnect --protocol=gp -u username --passwd-on-stdin xxxx:44333

# 证书连接
[root@dev ~]# echo -n password | openconnect -u username --passwd-on-stdin xxxx:44333
```



