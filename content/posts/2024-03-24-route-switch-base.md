+++
title = 'Route&Swich 基础'
date = 2024-03-23T21:16:02+08:00
draft = false
tags = ["network"]
categories = ["Network"]

+++
## 路由交换基础配置

设备名修改

```toml
<Huawei> system-view
[Huawei] sysname Server 
[Server] undo sysname 
[Huawei]
```

查看配置

```toml
# 查看当前配置
display this

# 查看当前所有配置
display current-configuration


```



禁用ftp功能

```toml
<Huawei> system-view
[Huawei] ftp server enable 
[Huawei] undo ftp server
```

删除port版定的ip

```toml
[Huawei]interface g0/0/1
[Huawei-GigabitEthernet0/0/1]ip address 192.168.1.1 24
[Huawei-GigabitEthernet0/0/1]undo ip address
```

配置时间

```toml
display clock  查看当前时间
clock timezone  设置所在时区
clock datetime 设置当前时间和日期
clock daylight-saving-time 设置采用夏时制
```

配置标题消息

```toml
# 配置登录前显示的标题信息
[Huawei] header login information "welcome to huawei"

# 配置登录后显示的标题信息
[Huawei] header shell information "Please do not reboot the device"
```

配置路由器接口设置IP地址

```toml
<Huawei> system-view
[Huawei] intface GigabitEthernet0/0/1
[Huawei-GigabitEthernet0/0/1] ip address 192.168.10.2 255.255.255.0
[Huawei-GigabitEthernet0/0/1]display ip interface brief  # 显示当前接口信息
******
Interface                         IP Address/Mask      Physical   Protocol  
GigabitEthernet0/0/1              192.168.10.2/24      down       down 

[Huawei-GigabitEthernet0/0/1]display this  # 显示当前配置
```

Telnet配置

```toml
# 开启telnet 服务
telnet server enable

# 查看状态
display telnet server status

# 进入VTY配置模式
user-interface vty 0 4

# 配置支持Telnet或SSH协议
protocol inbound telnet/ssh

# 配置认证模式
authentication-mode password/aaa

# 配置认证密码
set authentication password cipher huawei
# 配置用户级别
user privilege level 15

# 配置最大VTY会话数量
user-interface maximum-vty 15

# 进入AAA配置模式
aaa

# 创建用户和密码
local-user wakin password cipher huawei

# 配置用户级别
local-user wakin privilege level 15

# 配置用户可用服务
local-user wakin service-type telnet

#查看用户界面占用情况:
display users 

```

```toml
<Huawei>system-view
[Huawei]telnet server enable
[Huawei]aaa
[Huawei-aaa]loc-user huawei password irreversible-cipher Huawei@123
[Huawei-aaa]local-user huawei privilege level 15
[Huawei-aaa]local-user huawei service-type telnet
[Huawei-aaa]quit
[Huawei]user-interface vty 0 4
[Huawei]authentication-mode aaa

```



路由设置

```toml
# 显示路由表
display ip routing-table

# 静态路由
# ip route-static 目标网络 掩码 下一跳
ip route-static 192.168.1.0 24 10.1.12.2

# 缺省路由
# ip route-static 0.0.0.0 下一跳地址
ip route-static 0.0.0.0 10.1.12.3

# 浮动静态路由：通过调整优先级，实现路由的备份（即：主备备份）。
[RTA]ip route-static 20.0.0.0 30 10.1.1.2
[RTA]ip route-static 20.0.0.0 30 10.1.2.2 preference 70

# 防止路由环回
[RTB]ip route-static 10.1.0.0 16 null0
```



开启ssh

```toml
[R1]public-key local create rsa 
The range of public key modulus is (512 ~ 2048). 
If the key modulus is greater than 512, it will take a few minutes.
Press CTRL+C to abort.
Input the modulus length [default = 1024]:
Generating Keys...
..
Create the key pair successfully.

[R1]ssh server enable

# 进入 VTY 视图，配置验证模式为 scheme，设置用户权限为 Level-15，并配置协议为 ssh
[R1]user-interface vty 0 4
[R1-line-vty0-4]authentication-mode scheme
[R1-line-vty0-4]user-role level-15 
[R1-line-vty0-4]protocol inbound ssh

# 创建用于登录验证的用户，配置密码，配置用户权限为 Level-15，服务类型为 ssh
[R1]local-user wangdaye class manage 
New local user added.
[R1-luser-manage-wangdaye]password simple 123456
[R1-luser-manage-wangdaye]service-type ssh
[R1-luser-manage-wangdaye]authorization-attribute user-role level-15
```





### vlan

```toml
# 传教vlan
[SW1]vlan 10
[SW1-vlan10]vlan 20

# 把交换机的口加到vlan中
[SW1]int g1/0/2
[SW1-GigabitEthernet1/0/2]port access vlan 10
[SW1]int g1/0/3
[SW1-GigabitEthernet1/0/3]port access vlan 20

# 设置trunk口
[SW1]int g1/0/1
[SW1-GigabitEthernet1/0/1]port link-type trunk 
[SW1-GigabitEthernet1/0/1]port trunk permit vlan 10 20
```

单臂路由

```toml
[SW1]vlan 10
[SW1]vlan 20

[SW1]int g1/0/2
[SW1-GigabitEthernet1/0/2]port access vlan 10

[SW1]int g1/0/3
[SW1-GigabitEthernet1/0/3]port access vlan 20



[SW1]int g1/0/1
[SW1-GigabitEthernet1/0/1]port link-type trunk 
[SW1-GigabitEthernet1/0/1]port trunk permit vlan 10 20



[R1]int g0/0.1
[R1-GigabitEthernet0/0.1]vlan-type dot1q vid 10
[R1-GigabitEthernet0/0.1]ip add 192.168.1.254 24

[R1]int g0/0.2
[R1-GigabitEthernet0/0.2]vlan-type dot1q vid 20
[R1-GigabitEthernet0/0.2]ip add 192.168.2.254 24




[R1]dis ip routing-table 

Destinations : 16	Routes : 16

Destination/Mask   Proto   Pre Cost        NextHop         Interface
0.0.0.0/32         Direct  0   0           127.0.0.1       InLoop0
127.0.0.0/8        Direct  0   0           127.0.0.1       InLoop0
127.0.0.0/32       Direct  0   0           127.0.0.1       InLoop0
127.0.0.1/32       Direct  0   0           127.0.0.1       InLoop0
127.255.255.255/32 Direct  0   0           127.0.0.1       InLoop0
192.168.1.0/24     Direct  0   0           192.168.1.254   GE0/0.1
192.168.1.0/32     Direct  0   0           192.168.1.254   GE0/0.1
192.168.1.254/32   Direct  0   0           127.0.0.1       InLoop0
192.168.1.255/32   Direct  0   0           192.168.1.254   GE0/0.1
192.168.2.0/24     Direct  0   0           192.168.2.254   GE0/0.2
192.168.2.0/32     Direct  0   0           192.168.2.254   GE0/0.2
192.168.2.254/32   Direct  0   0           127.0.0.1       InLoop0
192.168.2.255/32   Direct  0   0           192.168.2.254   GE0/0.2
224.0.0.0/4        Direct  0   0           0.0.0.0         NULL0
224.0.0.0/24       Direct  0   0           0.0.0.0         NULL0
255.255.255.255/32 Direct  0   0           127.0.0.1       InLoop0
[R1]
```



三层交换机做路由

```toml
[SW1]vlan 10
[SW1-vlan10]vlan 20

[SW1]int g1/0/1
[SW1-GigabitEthernet1/0/1]port access vlan 10

[SW1]int g1/0/2
[SW1-GigabitEthernet1/0/2]port access vlan 20



[SW1]int vlan 10
[SW1-Vlan-interface10]ip add 192.168.1.254 24

[SW1]int vlan 20
[SW1-Vlan-interface20]ip add 192.168.2.254 24




[SW1]dis ip routing-table 

Destinations : 16	Routes : 16

Destination/Mask   Proto   Pre Cost        NextHop         Interface
0.0.0.0/32         Direct  0   0           127.0.0.1       InLoop0
127.0.0.0/8        Direct  0   0           127.0.0.1       InLoop0
127.0.0.0/32       Direct  0   0           127.0.0.1       InLoop0
127.0.0.1/32       Direct  0   0           127.0.0.1       InLoop0
127.255.255.255/32 Direct  0   0           127.0.0.1       InLoop0
192.168.1.0/24     Direct  0   0           192.168.1.254   Vlan10
192.168.1.0/32     Direct  0   0           192.168.1.254   Vlan10
192.168.1.254/32   Direct  0   0           127.0.0.1       InLoop0
192.168.1.255/32   Direct  0   0           192.168.1.254   Vlan10
192.168.2.0/24     Direct  0   0           192.168.2.254   Vlan20
192.168.2.0/32     Direct  0   0           192.168.2.254   Vlan20
192.168.2.254/32   Direct  0   0           127.0.0.1       InLoop0
192.168.2.255/32   Direct  0   0           192.168.2.254   Vlan20
224.0.0.0/4        Direct  0   0           0.0.0.0         NULL0
224.0.0.0/24       Direct  0   0           0.0.0.0         NULL0
255.255.255.255/32 Direct  0   0           127.0.0.1       InLoop0
[SW1]
```



DHCP

```toml
# 开启dhcp
[R1]dhcp enable

[R1]dhcp server ip-pool 1
[R1-dhcp-pool-1]network 192.168.1.0 mask 255.255.255.0
[R1-dhcp-pool-1]gateway-list 192.168.1.254
[R1-dhcp-pool-1]dns-list 202.103.24.68 202.103.0.117

# 设置专用地址段，要求不能用于自动分配
[R1]dhcp server forbidden-ip 192.168.1.10 192.168.1.20
```

