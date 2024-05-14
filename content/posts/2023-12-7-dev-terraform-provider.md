+++

title = 'Gitlab'
date = 2023-12-04T20:26:27+08:00
draft = false

tags = ["terraform-dev"]
categories = ["DevOps"]

+++

# Dev terraform provider

基于新框架tpf开发

开发示例：

* https://github.com/hashicorp/terraform-provider-scaffolding-framework
* https://github.com/serialt/terraform-provider-demo

参考示例：

* https://github.com/hashicorp/terraform-provider-hashicups



1、debug terraform

```
# makefile
default: install

build:
        go build -v ./...

install: 
        go install -v ./...
```



vscode 调试 terraform provider

1）vscode launch

```
cat .vscode/launch.json 
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Debug Terraform Provider",
            "type": "go",
            "request": "launch",
            "mode": "debug",
            // this assumes your workspace is the root of the repo
            "program": "${workspaceFolder}",
            "env": {},
            "args": [
                "-debug",
            ]
        }
    ]
}
```

run debug

2）ready terraform code 



3）run terraform

```
TF_REATTACH_PROVIDERS='{"registry.terraform.io/serialt/message":{"Protocol":"grpc","ProtocolVersion":5,"Pid":50972,"Test":true,"Addr":{"Network":"unix","String":"/var/folders/vm/zlhwbdyj2031f088_w9q3f8w0000gn/T/plugin3117659601"}}}' terraform apply
```





run terraform code in local

provider

```
cat ~/.terraformrc
provider_installation {
    dev_overrides {
      "serialt/message" = "/Users/serialt/go/bin"
    }
    direct {}
  }
```

```
# TF_LOG=TRACE terraform apply
terraform apply
```

