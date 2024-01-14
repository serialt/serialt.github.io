+++

title = 'Terraform Registry'
date = 2024-01-14T20:26:27+08:00
draft = false

tags = ["terraform-registry"]
categories = ["DevOps"]

+++

# Terraform Registry



## 一、Registry 协议

文档地址：https://developer.hashicorp.com/terraform/internals/provider-registry-protocol

```shell
# list可用的provider
curl 'https://registry.terraform.io/v1/providers/hashicorp/random/versions'

{
  "versions": [
    {
      "version": "2.0.0",
      "protocols": ["4.0", "5.1"],
      "platforms": [
        {"os": "darwin", "arch": "amd64"},
        {"os": "linux", "arch": "amd64"},
        {"os": "linux", "arch": "arm"},
        {"os": "windows", "arch": "amd64"}
      ]
    },
    {
      "version": "2.0.1",
      "protocols": ["5.2"],
      "platforms": [
        {"os": "darwin", "arch": "amd64"},
        {"os": "linux", "arch": "amd64"},
        {"os": "linux", "arch": "arm"},
        {"os": "windows", "arch": "amd64"}
      ]
    }
  ]
}



# find provider package
curl 'https://registry.terraform.io/v1/providers/hashicorp/random/2.0.0/download/linux/amd64'


{
  "protocols": ["4.0", "5.1"],
  "os": "linux",
  "arch": "amd64",
  "filename": "terraform-provider-random_2.0.0_linux_amd64.zip",
  "download_url": "https://releases.hashicorp.com/terraform-provider-random/2.0.0/terraform-provider-random_2.0.0_linux_amd64.zip",
  "shasums_url": "https://releases.hashicorp.com/terraform-provider-random/2.0.0/terraform-provider-random_2.0.0_SHA256SUMS",
  "shasums_signature_url": "https://releases.hashicorp.com/terraform-provider-random/2.0.0/terraform-provider-random_2.0.0_SHA256SUMS.sig",
  "shasum": "5f9c7aa76b7c34d722fc9123208e26b22d60440cb47150dd04733b9b94f4541a",
  "signing_keys": {
    "gpg_public_keys": [
      {
        "key_id": "51852D87348FFC4C",
        "ascii_armor": "-----BEGIN PGP PUBLIC KEY BLOCK-----\nVersion: GnuPG v1\n\nmQENBFMORM0BCADBRyKO1MhCirazOSVwcfTr1xUxjPvfxD3hjUwHtjsOy/bT6p9f\nW2mRPfwnq2JB5As+paL3UGDsSRDnK9KAxQb0NNF4+eVhr/EJ18s3wwXXDMjpIifq\nfIm2WyH3G+aRLTLPIpscUNKDyxFOUbsmgXAmJ46Re1fn8uKxKRHbfa39aeuEYWFA\n3drdL1WoUngvED7f+RnKBK2G6ZEpO+LDovQk19xGjiMTtPJrjMjZJ3QXqPvx5wca\nKSZLr4lMTuoTI/ZXyZy5bD4tShiZz6KcyX27cD70q2iRcEZ0poLKHyEIDAi3TM5k\nSwbbWBFd5RNPOR0qzrb/0p9ksKK48IIfH2FvABEBAAG0K0hhc2hpQ29ycCBTZWN1\ncml0eSA8c2VjdXJpdHlAaGFzaGljb3JwLmNvbT6JATgEEwECACIFAlMORM0CGwMG\nCwkIBwMCBhUIAgkKCwQWAgMBAh4BAheAAAoJEFGFLYc0j/xMyWIIAIPhcVqiQ59n\nJc07gjUX0SWBJAxEG1lKxfzS4Xp+57h2xxTpdotGQ1fZwsihaIqow337YHQI3q0i\nSqV534Ms+j/tU7X8sq11xFJIeEVG8PASRCwmryUwghFKPlHETQ8jJ+Y8+1asRydi\npsP3B/5Mjhqv/uOK+Vy3zAyIpyDOMtIpOVfjSpCplVRdtSTFWBu9Em7j5I2HMn1w\nsJZnJgXKpybpibGiiTtmnFLOwibmprSu04rsnP4ncdC2XRD4wIjoyA+4PKgX3sCO\nklEzKryWYBmLkJOMDdo52LttP3279s7XrkLEE7ia0fXa2c12EQ0f0DQ1tGUvyVEW\nWmJVccm5bq25AQ0EUw5EzQEIANaPUY04/g7AmYkOMjaCZ6iTp9hB5Rsj/4ee/ln9\nwArzRO9+3eejLWh53FoN1rO+su7tiXJA5YAzVy6tuolrqjM8DBztPxdLBbEi4V+j\n2tK0dATdBQBHEh3OJApO2UBtcjaZBT31zrG9K55D+CrcgIVEHAKY8Cb4kLBkb5wM\nskn+DrASKU0BNIV1qRsxfiUdQHZfSqtp004nrql1lbFMLFEuiY8FZrkkQ9qduixo\nmTT6f34/oiY+Jam3zCK7RDN/OjuWheIPGj/Qbx9JuNiwgX6yRj7OE1tjUx6d8g9y\n0H1fmLJbb3WZZbuuGFnK6qrE3bGeY8+AWaJAZ37wpWh1p0cAEQEAAYkBHwQYAQIA\nCQUCUw5EzQIbDAAKCRBRhS2HNI/8TJntCAClU7TOO/X053eKF1jqNW4A1qpxctVc\nz8eTcY8Om5O4f6a/rfxfNFKn9Qyja/OG1xWNobETy7MiMXYjaa8uUx5iFy6kMVaP\n0BXJ59NLZjMARGw6lVTYDTIvzqqqwLxgliSDfSnqUhubGwvykANPO+93BBx89MRG\nunNoYGXtPlhNFrAsB1VR8+EyKLv2HQtGCPSFBhrjuzH3gxGibNDDdFQLxxuJWepJ\nEK1UbTS4ms0NgZ2Uknqn1WRU1Ki7rE4sTy68iZtWpKQXZEJa0IGnuI2sSINGcXCJ\noEIgXTMyCILo34Fa/C6VCm2WBgz9zZO8/rHIiQm1J5zqz0DrDwKBUM9C\n=LYpS\n-----END PGP PUBLIC KEY BLOCK-----",
        "trust_signature": "",
        "source": "HashiCorp",
        "source_url": "https://www.hashicorp.com/security.html"
      }
    ]
  }
}


```



## 二、Cache Terraform Registry

​		在执行`terraform init `的时候，`terraform cli` 会从官方的`Registry`查找和下载对应的`provider`，小的provider几M，大的可能几百M，而且如果当执行`plan`或`apply`出现问题需要重新执行`init`，可能需要多次请求`registry`，这对网络的速度和稳定性要求比较高。因此，基于官方的 Provider Registry Protocol 和 Remote Service，可以开发一个简单的cli缓存和提供Registry服务。

​		

### 1、terraform下载provider原理

​		当执行`terraform init `后，`terraform cli` 会根据 tf文件中定义的`provider`去向 `registry.terraform.io` 发起`GET`请求，例如：tf文件中有使用`hashicorp/random`，`terraform cli` 会发出Get请求 `https://registry.terraform.io/v1/providers/hashicorp/random/versions`，请求到数据后再跟tf文件中定义的版本判断一致后获取要下载的`provider package` 包的信息，例如：`https://registry.terraform.io/v1/providers/hashicorp/random/2.0.0/download/linux/amd64`，返回的数据中有provider的下载地址、provide的hash值和用于验证provider签名的公钥，由`hashicorp`维护的`provider package`会存放在`releases.hashicorp.com`中，而由第三方开发者维护的只能使用github托管，要求代码必须是开源的，且repo的命名格式符合 `terraform-provider-ssh`。



### 2、缓存加速原理

`~/.terraformrc`文件中可以可以配置镜像，例如：

```shell
host "registry.terraform.io" {
  services = {
    "modules.v1" = "http://127.0.0.1:9090/pub/v1/modules/",
    "providers.v1" = "http://127.0.0.1:9090/v1/providers/"
  }
}
```

当执行`terraform init`的时候，访问 `registry.terraform.io/v1/providers/` 会被替换为`http://127.0.0.1:9090/v1/providers/`



## 三、基于nexus存储provider

### 1、下载数据并同步到nexus

配置文件格式

```yaml
storage:
  type: nexus
  nexus:
    url: http://nexus.local.com
    repo: terraform
tf:
  providerLast: 2
  provider:
    - "hashicorp/random"
    - "hashicorp/aws"
    - "hashicorp/tls"
    - "hashicorp/dns"
  providerOS:
    - linux
    - darwin
  providerArch:
    - amd64
    - arm64
  providerVersion:
    hashicorp/random:
      - "3.6.0"
      - "3.3.2"
    loafoe/ssh:
      - "2.5.0"
```

```shell
# 获取provider的版本，把这个文件上传到nexus，路径: hashicorp/random/versions
curl 'https://registry.terraform.io/v1/providers/hashicorp/random/versions'

# 获取到版本信息后请求下载的provider package 信息 ,下载里面的文件，并把对应的链接信修改为nexus中的地址
curl  https://registry.terraform.io/v1/providers/hashicorp/random/2.0.0/download/linux/amd64'


# 下载上一步获取到的原始 provider pcakge 链接中的文件，把文件上传到nexus
```



### 2、实现Provider Registry Protocol

根据 `Provider Registry Protocol`实现一个私有的`registry`，根据请求来的信息去请求`nexus`上的对应数据，把数据返回给`terraform cli `, `terraform cli` 会自己从接收到的数据去从nexus下载对应版本的provider。

