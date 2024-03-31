+++
title = 'Publish Provider to Registry'
date = 2024-03-31T16:16:02+08:00
draft = false
tags = ["terraform-registry","hashicorp"]
categories = ["DevOps"]

+++
## Publish Terraform Provider

https://registry.terraform.io/ 上的provider只能托管在 github 上



### 1、Terraform Registry 官网上注册账号

账号使用 github 登录，设置 github 需要对 terraform registry 的授权

### 2、创建 github project，配置 github action

名字需要符合 terraform-provider-xxxxx，例如 terraform-provider-message，

1）生成 GPG key，用于 provider 签名和验签

```shell
[root@dev ~]# gpg --full-generate-key
gpg (GnuPG) 2.2.20; Copyright (C) 2020 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Please select what kind of key you want:
   (1) RSA and RSA (default)
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
  (14) Existing key from card
Your selection? 1   # 选择RSA
RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (2048) 4096   # 输入加密密钥的长度，4096
Requested keysize is 4096 bits
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 0    # 设置密码有效期
Key does not expire at all
Is this correct? (y/N) y

GnuPG needs to construct a user ID to identify your key.

Real name: serialt        # 设置密钥的消息
Email address: t@local.com
Comment: msg
You selected this USER-ID:
    "serialt (msg) <t@local.com>"

Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? o
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
gpg: key A02032881C9460CE marked as ultimately trusted
gpg: revocation certificate stored as '/root/.gnupg/openpgp-revocs.d/16630E5129BFAB582DDFBF71A02032881C9460CE.rev'
public and secret key created and signed.

pub   rsa4096 2021-08-23 [SC]
      16630E5129BFAB582DDFBF71A02032881C9460CE
uid                      serialt (msg) <t@local.com>
sub   rsa4096 2021-08-23 [E]




        ┌──────────────────────────────────────────────────────┐
        │ Please enter the passphrase to                       │
        │ protect your new key                                 │
        │                                                      │
        │ Passphrase: ________________________________________ │
        │                                                      │
        │       <OK>                              <Cancel>     │
        └──────────────────────────────────────────────────────┘








         ┌──────────────────────────────────────────────────────┐
         │ Please re-enter this passphrase                      │
         │                                                      │
         │ Passphrase: ________________________________________ │
         │                                                      │
         │       <OK>                              <Cancel>     │
         └──────────────────────────────────────────────────────┘


# 查看所有密钥
gpg -k

# 导出密钥
gpg --output public_key.gpg --armor --export 375xxxxx
gpg --output private_key.gpg --armor --export-secret-key 375xxxxx
```



2）github action

需要配置一下 secret

* TOKEN
* GPG_PRIVATE_KEY

```yaml
# Terraform Provider release workflow.
name: Release

# This GitHub action creates a release when a tag that matches the pattern
# "v*" (e.g. v0.1.0) is created.
on:
  push:
    tags:
      - 'v*'

# Releases need permissions to read and write the repository contents.
# GitHub considers creating releases and uploading assets as writing contents.
permissions:
  contents: write

jobs:
  goreleaser:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@b4ffde65f46336ab88eb53be808477a3936bae11 # v4.1.1
        with:
          # Allow goreleaser to access older tag information.
          fetch-depth: 0
      - uses: actions/setup-go@0c52d547c9bc32b1aa3301fd7a9cb496313a4491 # v5.0.0
        with:
          go-version-file: 'go.mod'
          cache: true
      - name: Import GPG key
        uses: crazy-max/ghaction-import-gpg@01dd5d3ca463c7f10f7f4f7b4f177225ac661ee4 # v6.1.0
        id: import_gpg
        with:
          gpg_private_key: ${{ secrets.GPG_PRIVATE_KEY }}
          #passphrase: ${{ secrets.GPG_PRIVATE_PASSPHRASE }}      
      - name: Run GoReleaser
        uses: goreleaser/goreleaser-action@7ec5c2b0c6cdda6e8bbb49444bc797dd33d74dd8 # v5.0.0
        with:
          args: release --clean
        env:
          # GitHub sets the GITHUB_TOKEN secret automatically.
          GITHUB_TOKEN: ${{ secrets.TOKEN }}
          GPG_FINGERPRINT: ${{ steps.import_gpg.outputs.fingerprint }}
```

推送项目到github



### 3、Terraform Registry上配置公钥和发布provider

1）导入公钥 https://registry.terraform.io/settings/gpg-keys

2）发布provider https://registry.terraform.io/publish/provider