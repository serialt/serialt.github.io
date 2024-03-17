+++
title = 'Terraform Registry Mirror'
date = 2024-03-17T18:37:07+08:00
draft = false

tags = ["terraform-registry","hashicorp"]
categories = ["DevOps"]

+++
## Terraform Registry Mirror

terraform 的 provider 托管在海外，使用时候会经常因为网络问题导致下载失败，terraform 官方有 Provider Network Mirror Protocol，支持使用本地文件和 Static Website，制作方法都依赖于`terraform providers mirror `命令获取到的资源。

### 1、制作离线 provider 资源

#### 在当前目录下生成 provider.tf 文件

```
terraform {
  required_providers {
    docker = {
      source  = "kreuzwerker/docker"
      version = "2.20.3"
    }
  }

}
```

#### 创建存储的目录，并下载 provider

```shell
# 下载当前版 terraform 架构的 provider 到 provider 目录，可以用 -platform=OS_ARCH 指定架构
[serialt@Sugar tf-demo]🐳 mkdir providers
terraform providers mirror providers 

# 下载 darwin_arm64 linux_amd64 
[serialt@Sugar tf-demo]🐳 terraform providers mirror -platform=linux_amd64 providers 
[serialt@Sugar tf-demo]🐳 terraform providers mirror -platform=darwin_arm64 providers 

[serialt@Sugar tf-demo]🐳 tree providers
providers
└── registry.terraform.io
    └── kreuzwerker
        └── docker
            ├── 2.20.3.json
            ├── index.json
            ├── terraform-provider-docker_2.20.3_darwin_arm64.zip
            └── terraform-provider-docker_2.20.3_linux_amd64.zip

3 directories, 4 files
```



### 2、本地文件镜像

配置插件缓存目录

```shell
export TF_PLUGIN_CACHE_DIR=/home/user/.terraform.d/plugin-cache
```

配置` ~/.terraformrc`

```shell
provider_installation {
  filesystem_mirror {
    path    = "/home/user/tf-demo/providers"
    include = ["*/*/*"]
  }
  direct {
    exclude = ["*/*/*"]
  }
}
```

```shell
[serialt@Sugar tf-ccc]🐳 terraform init

Initializing the backend...

Initializing provider plugins...
- Finding kreuzwerker/docker versions matching "2.20.3"...
- Installing kreuzwerker/docker v2.20.3...
- Installed kreuzwerker/docker v2.20.3 (unauthenticated)

Terraform has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.

╷
│ Warning: Incomplete lock file information for providers
│ 
│ Due to your customized provider installation methods, Terraform was forced to
│ calculate lock file checksums locally for the following providers:
│   - kreuzwerker/docker
│ 
│ The current .terraform.lock.hcl file only includes checksums for
│ darwin_arm64, so Terraform running on another platform will fail to install
│ these providers.
│ 
│ To calculate additional checksums for another platform, run:
│   terraform providers lock -platform=linux_amd64
│ (where linux_amd64 is the platform to generate)
╵

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```



### 3、网络镜像

#### nginx配置文件

```nginx
server {
        server_name tf-mirror.local.com;
        listen 80;
        return 301 https://$server_name$request_uri;
}

server{
        server_name tf-mirror.local.com;
        listen 443 ssl;
        charset utf-8;
        gzip on;
        gzip_min_length 1K;
        gzip_comp_level 4;
        gzip_buffers 32 4K;
        gzip_types text/plain application/javascript application/x-javascript text/css application/xml text/javascript application/x-httpd-php image/jpeg image/gif image/png;
        gzip_vary on;
        add_header Strict-Transport-Security "max-age=31536000; includeSubDomians;proload" always;
        root /data/terraform-mirror/;
        index index.html index.htm;
        try_files $uri $uri/ /index.html;
        client_max_body_size  20m;
        access_log /var/log/nginx/tf-mirror.local.com.log main;
        error_log /var/log/nginx/tf-mirror.local.com-err.log;
        ssl_certificate /etc/nginx/cert_files/local.com.crt;
        ssl_certificate_key /etc/nginx/cert_files/local.com.key;
}
```

#### 上传下载的 provider 资源到 web 根目录

例如：`/data/terraform-mirror/`

```shell
[serialt@Sugar tf-ccc]🐳 scp -r ./providers  local.com:/data/terraform-mirror/
```

启动nginx服务

配置`~/.terraformrc`文件

```shell
provider_installation {
  network_mirror {
    url = "https://tf-mirror.local.com/providers/"
  }
}
```

```shell
[serialt@Sugar tf-ccc]🐳 terraform init

Initializing the backend...

Initializing provider plugins...
- Finding latest version of hashicorp/random...
- Installing hashicorp/random v3.6.0...
- Installed hashicorp/random v3.6.0 (verified checksum)

Terraform has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```



### 4、基于脚本下载 provider 离线资源

* https://github.com/serialt/terraform-registry-mirror

要求：

* terraform
* jq

1）修改下载的平台和架构

```shell
sed -ri PLATFORMS="linux_amd64 darwin_amd64 darwin_arm64"

sed -ri '/^PLATFORMS=/cPLATFORMS="linux_amd64 darwin_amd64 darwin_arm64"' bin/mirror.sh
sed -ri '/^PLATFORMS=/cPLATFORMS="linux_amd64 darwin_amd64 darwin_arm64"' bin/core.sh
```

2）下载 terraform

```
bin/core.sh terraform.json
```

3）下载 provider

```shell
# 下载单个
bin/core.sh ./providers/aws.json


# 下载多个
mirror(){
    action=$1
    providers=`ls ./providers`
    for i in ${providers};do 
        bash ./bin/${action} ./providers/${i}
    done
}

mirror mirror.sh
# mirror updater.sh
```

