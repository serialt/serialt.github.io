+++
title = 'Ubuntu cloud-init'
date = 2024-10-26T14:38:01+08:00
draft = false

tags = ["vm","ubuntu"]
categories = ["Ubuntu 自动安装"]

+++
cloud-init 

https://hmli.ustc.edu.cn/doc/linux/ubuntu-autoinstall/ubuntu-autoinstall.html



ubuntu 安装后获取当前的user-data: /var/log/installer/autoinstall-user-data 



示例：

```yaml
#cloud-config
autoinstall:
  version: 1
  apt:
    disable_components: []
    fallback: offline-install
    geoip: true
    mirror-selection:
      primary:
      - uri: http://mirrors.ustc.edu.cn/ubuntu-ports
      - country-mirror
      - arches: &id001
        - amd64
        - i386
        uri: http://mirrors.ustc.edu.cn/ubuntu/
      - arches: &id002
        - s390x
        - arm64
        - armhf
        - powerpc
        - ppc64el
        - riscv64
        uri: http://mirrors.ustc.edu.cn/ubuntu-ports
    preserve_sources_list: false
    security:
    - arches: *id001
      uri: http://mirrors.ustc.edu.cn/ubuntu/
    - arches: *id002
      uri: http://mirrors.ustc.edu.cn/ubuntu-ports
  codecs:
    install: false
  drivers:
    install: false
  identity:
    hostname: localhost
    password: $6$4DsNTlBjlChIXq/O$AgJbtZDu4g50qcDjx5NUjgkSen2nNsOVo/aeo7U5On8NpqHEquCVFCjlmjlVBCVCs6prGev8TYlZG/FDUjKd..
    realname: sugar
    username: sugar
  kernel:
    package: linux-generic
  keyboard:
    layout: us
    toggle: null
    variant: ''
  locale: en_US.UTF-8
  user-data:
    timezone: Asia/Shanghai
    # disable_root: false
  network:
    version: 2
    ethernets:
      eth:
        match:
          name: "en*"
        dhcp4: yes
  packages:
    - net-tools
    - vim
  oem:
    install: auto
  source:
    id: ubuntu-server-minimal
    search_drivers: false
  ssh:
    allow-pw: true
    authorized-keys: []
    install-server: true
  storage:
    layout:
      name: lvm

  updates: security
  late-commands:
      - echo 'sugar ALL=(ALL) NOPASSWD:ALL' > /target/etc/sudoers.d/sugar
```





