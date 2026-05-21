+++

title = 'TF-dev'
date = 2026-05-21T20:26:27+08:00
draft = false

tags = ["terraform-dev","tf-dev"]
categories = ["DevOps"]

+++

# Dev terraform provider

基于新框架tpf开发

开发示例：

* https://github.com/hashicorp/terraform-provider-scaffolding-framework
* https://github.com/serialt/terraform-provider-demo
* https://github.com/serialt/terraform-provider-dnshe

参考示例：

* https://github.com/hashicorp/terraform-provider-hashicups

### 一、调试 provider

#### 1、debug terraform

```makefile
# Makefile
default: fmt lint install generate

build:
	go build -v ./...

install: build
	go install -v ./...

lint:
	golangci-lint run

generate:
	cd tools; go generate ./...

fmt:
	gofmt -s -w -e .

test:
	go test -v -cover -timeout=120s -parallel=10 ./...

testacc:
	TF_ACC=1 go test -v -cover -timeout 120m ./...

.PHONY: fmt lint test testacc build install generate

```



vscode 调试 terraform provider

##### 1）vscode launch

```json
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
    },
    {
      "name": "Launch test function",
      "type": "go",
      "request": "launch",
      "mode": "test",
      "program": "${fileDirname}",
      "showLog": true,
      "args": [
        "-test.v",
        "-test.run",
        "TestDatasource_RepositoryDockerHosted"
      ]
    }
  ]
}
```

run debug

##### 2）ready terraform code

##### 3）run terraform

```
TF_REATTACH_PROVIDERS='{"registry.terraform.io/serialt/message":{"Protocol":"grpc","ProtocolVersion":5,"Pid":50972,"Test":true,"Addr":{"Network":"unix","String":"/var/folders/vm/zlhwbdyj2031f088_w9q3f8w0000gn/T/plugin3117659601"}}}' terraform apply
```

##### 4）点击测试方法调试debug

设置环境变量

```shell
cat .vscode/launch.json 
{
    "go.toolsEnvVars": {
        "TF_LOG": "DEBUG",
        "TF_ACC": "1",
        "DNSHE_API_KEY": "cfxxxxx",
        "DNSHE_API_SECRET": "e3axxxxxx"
    },
}
```



##### 5) dev模式覆盖

方便本地执行terraform init

```bash
cat ~/.terraformrc
provider_installation {
    dev_overrides {
      "serialt/message" = "/Users/krab/go/bin"
      "serialt/dnshe" = "/Users/krab/go/bin"
    }
    direct {}
    // network_mirror {
    // 	url = "https://mirrors.local.io/public/mirror/"
    // }
  }
```

```
# TF_LOG=TRACE terraform apply
terraform apply
```

##### 6)生成文档

引入tfplugindocs

```bash
[krab@Krab terraform-provider-dnshe]🐳 tree tools/
tools/
├── go.mod
├── go.sum
└── tools.go

1 directory, 3 files
```

`tools.go`

```go
//go:build generate

package tools

import (
	_ "github.com/hashicorp/copywrite"
	_ "github.com/hashicorp/terraform-plugin-docs/cmd/tfplugindocs"
)

// Format Terraform code for use in documentation.
// If you do not have Terraform installed, you can remove the formatting command, but it is suggested
// to ensure the documentation is formatted properly.
//go:generate terraform fmt -recursive ../examples/

// Generate documentation.
//go:generate go run github.com/hashicorp/terraform-plugin-docs/cmd/tfplugindocs generate --provider-dir .. -provider-name dnshe

```



生成文档

> 特殊说明： 以下示例结构代码会被填充到doc中
>
> examples/data-sources/dnshe_dns_quota/data-source.tf
>
> examples/resources/dnshe_dns_record/resource.tf
>
> examples/provider/provider.tf

```bash
# 
make generate
```



### 二、TPF框架

目前官方在维护的开发 provider sdk 有两个版本：

* terraform-plugin-sdk/v2
* terraform-plugin-framework

如果要支持老版本的terraform，建议是使用 terraform-plugin-sdk/v2 开发，否则还是建议使用terraform-plugin-framework开发。

TPF开发示例：https://github.com/hashicorp/terraform-provider-hashicups



#### 解释说明

插件开发主要涉及的文件分为三种类型：

* provider
* datasource
* resource

##### provider

文件中主要是一些 配置文件的读取和对第三分 api 客户端的初始化，主要由五个方法组成：

* Metadata：provider的名字
* Schema：provider的配置参数
* Configure：读取配置provider中的值和环境变量的值，初始化调用第三方api的client
* Resources：定义的resource资源
* DataSources：定义的datasource资源

##### datasource

通过第三方api的client调用，查询已存在的资源信息：

* Metadata：定义datasource的名字
* Schema：datasource配置的参数
* Read：client调用api，读取具体的数据，生成state
* Configure：从provider中获取传入的调用api的client

##### resource

通过第三方api的client调用，对资源进行增删改查和导入资源对象

* Metadata：定义resource的名字

* Schema：resource配置的参数
* Configure：从provider中获取传入的调用api的client
* Create：创建资源
* Read：查询资源
* Update：更新资源
* Delete：删除资源，用于在销毁时删除资源
* ImportState：导入资源，用terraform管理存量的资源对象。



main.go

```go
package main

import (
	"context"
	"flag"
	"log"

	"github.com/hashicorp/terraform-plugin-framework/providerserver"
	"github.com/serialt/terraform-provider-dnshe/internal/provider"
)

var (
	version string = "dev"
)

func main() {
	var debug bool

	flag.BoolVar(&debug, "debug", false, "set to true to run the provider with support for debuggers like delve")
	flag.Parse()

	opts := providerserver.ServeOpts{
		Address: "registry.terraform.io/serialt/dnshe",
		Debug:   debug,
	}

	err := providerserver.Serve(context.Background(), provider.New(version), opts)

	if err != nil {
		log.Fatal(err.Error())
	}
}

```

provider.go

```go
package provider

import (
	"context"
	"os"

	"github.com/hashicorp/terraform-plugin-framework/datasource"
	"github.com/hashicorp/terraform-plugin-framework/function"
	"github.com/hashicorp/terraform-plugin-framework/provider"
	"github.com/hashicorp/terraform-plugin-framework/provider/schema"
	"github.com/hashicorp/terraform-plugin-framework/resource"
	"github.com/hashicorp/terraform-plugin-framework/types"
	"github.com/serialt/terraform-provider-dnshe/dnshe"
)

var _ provider.Provider = &DNSHEProvider{}
var _ provider.ProviderWithFunctions = &DNSHEProvider{}

type DNSHEProvider struct {
	version string
}
type DNSHEProviderModel struct {
	BaseURL   types.String `tfsdk:"base_url"`
	ApiKey    types.String `tfsdk:"api_key"`
	ApiSecret types.String `tfsdk:"api_secret"`
}

func (p *DNSHEProvider) Metadata(ctx context.Context, req provider.MetadataRequest, resp *provider.MetadataResponse) {
	resp.TypeName = "dnshe"
	resp.Version = p.version
}

func (p *DNSHEProvider) Schema(ctx context.Context, req provider.SchemaRequest, resp *provider.SchemaResponse) {
	resp.Schema = schema.Schema{
		MarkdownDescription: "DNSHE Terraform Provider，用于全自动化管理自定义二级域名及 DNS 解析记录。",
		Attributes: map[string]schema.Attribute{
			"base_url": schema.StringAttribute{
				MarkdownDescription: "base_url",
				Optional:            true,
			},
			"api_key": schema.StringAttribute{
				MarkdownDescription: "api_key",
				Optional:            true,
			},
			"api_secret": schema.StringAttribute{
				MarkdownDescription: "api_secret",
				Optional:            true,
				Sensitive:           true,
			},
		},
	}
}

func (p *DNSHEProvider) Configure(ctx context.Context, req provider.ConfigureRequest, resp *provider.ConfigureResponse) {
	var data DNSHEProviderModel
	resp.Diagnostics.Append(req.Config.Get(ctx, &data)...)
	if resp.Diagnostics.HasError() {
		return
	}
	apiKey := os.Getenv("DNSHE_API_KEY")
	apiSecret := os.Getenv("DNSHE_API_SECRET")
	baseURL := data.BaseURL.ValueString()

	if !data.ApiKey.IsNull() {
		apiKey = data.ApiKey.ValueString()
	}
	if !data.ApiSecret.IsNull() {
		apiSecret = data.ApiSecret.ValueString()
	}

	if apiKey == "" || apiSecret == "" {
		resp.Diagnostics.AddError("凭证缺失", "必须配置 api_key 和 api_secret (或设置对应的环境变量)")
		return
	}

	client := dnshe.NewClient(baseURL, apiKey, apiSecret)
	resp.DataSourceData = client
	resp.ResourceData = client
}

func (p *DNSHEProvider) Resources(ctx context.Context) []func() resource.Resource {
	return []func() resource.Resource{
		NewDNSRecordResource,
		NewSubdomainResource,
	}
}

func (p *DNSHEProvider) DataSources(ctx context.Context) []func() datasource.DataSource {
	return []func() datasource.DataSource{
		NewDNSRecordsDataSource,
		NewQuotaDataSource,
		NewSubdomainDataSource,
		NewSubdomainsDataSource,
	}
}

func (p *DNSHEProvider) Functions(ctx context.Context) []func() function.Function {
	return []func() function.Function{}
}

func New(version string) func() provider.Provider {
	return func() provider.Provider {
		return &DNSHEProvider{
			version: version,
		}
	}
}

```

data_source.go

```go
package provider

import (
	"context"

	"github.com/hashicorp/terraform-plugin-framework/datasource"
	"github.com/hashicorp/terraform-plugin-framework/datasource/schema"
	"github.com/hashicorp/terraform-plugin-framework/types"
	"github.com/serialt/terraform-provider-dnshe/dnshe"
)

// ==========================================
// 4. 账号配额数据源 (dnshe_quota)
// ==========================================
type quotaDataSource struct{ client *dnshe.Client }

type quotaDSModel struct {
	ID          types.String `tfsdk:"id"`
	Used        types.Int64  `tfsdk:"used"`
	Base        types.Int64  `tfsdk:"base"`
	InviteBonus types.Int64  `tfsdk:"invite_bonus"`
	Total       types.Int64  `tfsdk:"total"`
	Available   types.Int64  `tfsdk:"available"`
}

func NewQuotaDataSource() datasource.DataSource { return &quotaDataSource{} }
func (d *quotaDataSource) Metadata(_ context.Context, req datasource.MetadataRequest, resp *datasource.MetadataResponse) {
	resp.TypeName = req.ProviderTypeName + "_dns_quota"
}
func (d *quotaDataSource) Configure(_ context.Context, req datasource.ConfigureRequest, resp *datasource.ConfigureResponse) {
	if req.ProviderData != nil {
		d.client = req.ProviderData.(*dnshe.Client)
	}
}
func (d *quotaDataSource) Schema(_ context.Context, _ datasource.SchemaRequest, resp *datasource.SchemaResponse) {
	resp.Schema = schema.Schema{
		Attributes: map[string]schema.Attribute{
			"id":           schema.StringAttribute{Computed: true},
			"used":         schema.Int64Attribute{Computed: true},
			"base":         schema.Int64Attribute{Computed: true},
			"invite_bonus": schema.Int64Attribute{Computed: true},
			"total":        schema.Int64Attribute{Computed: true},
			"available":    schema.Int64Attribute{Computed: true},
		},
	}
}
func (d *quotaDataSource) Read(ctx context.Context, req datasource.ReadRequest, resp *datasource.ReadResponse) {
	var data quotaDSModel
	res, err := d.client.GetQuota()
	if err != nil {
		resp.Diagnostics.AddError("API错误", err.Error())
		return
	}
	data.ID = types.StringValue("quota")
	data.Used = types.Int64Value(int64(res.Quota.Used))
	data.Base = types.Int64Value(int64(res.Quota.Base))
	data.InviteBonus = types.Int64Value(int64(res.Quota.InviteBonus))
	data.Total = types.Int64Value(int64(res.Quota.Total))
	data.Available = types.Int64Value(int64(res.Quota.Available))
	resp.Diagnostics.Append(resp.State.Set(ctx, &data)...)
}

```

data_source、resource、function 需要注册有才能使用

编写代码时，主要工作难点是 Schema 和接收schema的model



#### resource/schema 介绍，以schema.StringAttribute为例

```shell
CustomType  允许使用自定义属性类型来代替
Required   resource config 必填项
Optional resource config 可选项
Computed  resource config 只读
Sensitive  resource  config 敏感数据不输出
Description
MarkdownDescription 
DeprecationMessage   resource  config 弃用说明
Validators
PlanModifiers
Default
```



### 三、Terraform apply说明

1、apply，如果state文件中没有对应的资源则会触发create，create后会写入state文件

2、删除资源会先触发read，yes后触发删除操作

3、resource中定义的有改动导致资源某些配置会被replace，先触发read，yes 后会触发更新

4、create、update、read 最后都会获取资源状态数据并更新到state文件中



### 四、代码片段

```go
// schema
	DataSourceID = schema.StringAttribute{
		Description:         "Used to identify data source at nexus",
		MarkdownDescription: "Used to identify data source at nexus",
		Computed:            true,
	}

// 定义
// block
	resp.Schema = schema.Schema{
		Description:         "Use this data source .",
		MarkdownDescription: "Use this data source .",
		Blocks: map[string]schema.Block{
			"group":   tschema.DSGroup,
		},
		Attributes: map[string]schema.Attribute{
			"id":     tschema.DataSourceID,
			"name":   tschema.DataSourceName,
		},
	}

	DSGroup = schema.SingleNestedBlock{
		Description:         "The group configuration of the repository",
		MarkdownDescription: "The group configuration of the repository",
		Attributes: map[string]schema.Attribute{
			"member_names": schema.ListAttribute{
				Description:         "Member repositories names",
				MarkdownDescription: "Member repositories names",
				ElementType:         types.StringType,
				Required:            true,
				Validators:          []validator.List{listvalidator.SizeAtLeast(1)},
			},
		},
	}


// 定义list
// model 里面
type CleanupModel struct {
	PolicyNames types.List `tfsdk:"policy_names"`
}

// schema
	DSCleanUp = schema.SingleNestedBlock{
		MarkdownDescription: "Cleanup policies",
		Description:         "Cleanup policies",
		Attributes: map[string]schema.Attribute{
			"policy_names": schema.ListAttribute{
				Description:         "List of policy names",
				MarkdownDescription: "List of policy names",
				Computed:            true,
				ElementType:         types.StringType,
			},
		},
	}

// List 转 types.List 
	if repo.Cleanup != nil {
		policyNames := []attr.Value{}
		for _, item := range repo.Cleanup.PolicyNames {
			policyNames = append(policyNames, types.StringValue(item))
		}
		policyNamesTfsdk, _ := types.ListValue(types.StringType, policyNames)
		data.Cleanup = &model.CleanupModel{
			PolicyNames: policyNamesTfsdk,
		}
	}

// model 转 types.List
	actions := []string{}
	_ = plan.Actions.ElementsAs(ctx, actions, true)

// resource 设置了默认值后需要设置 Computed 为 true
			"v1_enabled": schema.BoolAttribute{
				Description:         "Whether to allow clients to use",
				MarkdownDescription: "Whether to allow clients to use",
				Computed:            true,
				Default:             booldefault.StaticBool(false),
			},



// 默认值
func (r *dnsRecordResource) Schema(_ context.Context, _ resource.SchemaRequest, resp *resource.SchemaResponse) {
	resp.Schema = schema.Schema{
		Attributes: map[string]schema.Attribute{
			"id": schema.StringAttribute{
				Computed:      true,
				PlanModifiers: []planmodifier.String{stringplanmodifier.UseStateForUnknown()},
			},
			"subdomain_id": schema.Int64Attribute{Required: true},
			"type":         schema.StringAttribute{Required: true},
			"name":         schema.StringAttribute{Optional: true, Computed: true},
			"content":      schema.StringAttribute{Required: true},
			"ttl":          schema.Int64Attribute{Optional: true, Computed: true, Default: int64default.StaticInt64(600)},
			"priority":     schema.Int64Attribute{Optional: true},
			"line":         schema.StringAttribute{Optional: true, Computed: true},
		},
	}
}

```

测试代码

```go
package provider

import (
	"fmt"
	"testing"

	"github.com/hashicorp/terraform-plugin-testing/helper/resource"
)

func TestAccDataSourceDNSQuota(t *testing.T) {
	resource.Test(t, resource.TestCase{
		PreCheck:                 func() { testAccPreCheck(t) },
		ProtoV6ProviderFactories: testAccProtoV6ProviderFactories,
		Steps: []resource.TestStep{
			{
				Config: testAccDataSourceDNSQuotaConfig,
				Check: resource.ComposeAggregateTestCheckFunc(
					// resource.TestCheckResourceAttr("data.dnshe_subdomain.test_domain", "subdomain", "krab"),
					// resource.TestCheckResourceAttrSet("data.dnshe_subdomain.test_domain", "id"),
					resource.TestCheckResourceAttrWith("data.dnshe_dns_quota.test_quota", "total", func(value string) error {
						fmt.Printf("\n======================= 🔍 DNSHE quota 🔍 =======================\n")
						fmt.Printf("  total 值为 : %s", value)

						return nil
					}),
				),
			},
		},
	})
}

const testAccDataSourceDNSQuotaConfig = `
data "dnshe_dns_quota" "test_quota" {
}
`





// testAccProtoV6ProviderFactories are used to instantiate a provider during
// acceptance testing. The factory function will be invoked for every Terraform
// CLI command executed to create a provider server to which the CLI can
// reattach.
var testAccProtoV6ProviderFactories = map[string]func() (tfprotov6.ProviderServer, error){
	"dnshe": providerserver.NewProtocol6WithError(New("test")()),
}

func testAccPreCheck(t *testing.T) {
	// You can add code here to run prior to any test case execution, for example assertions
	// about the appropriate environment variables being set are common to see in a pre-check
	// function.

	if os.Getenv("DNSHE_API_KEY") == "" {
		t.Fatal("环境缺失: 必须设置环境变量 DNSHE_API_KEY 才能运行集成测试")
	}
	if os.Getenv("DNSHE_API_SECRET") == "" {
		t.Fatal("环境缺失: 必须设置环境变量 DNSHE_API_SECRET 才能运行集成测试")
	}

}


```



### 五、发布provider

https://registry.terraform.io/ 上的provider目前只能托管在 github 上

#### 1、Terraform Registry 官网上注册账号

账号使用 github 登录，设置 github 需要对 terraform registry 的授权

#### 2、创建 github project，配置 github action

名字需要符合 terraform-provider-xxxxx，例如 terraform-provider-message，

#### 3、生成 GPG key，用于 provider 签名和验签

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



#### 4、github action

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

#### 5、Terraform Registry 发布provider

1）导入公钥 https://app.terraform.io/app/serialt/registry/public-namespaces/serialt/settings

2）发布provider https://app.terraform.io/app/serialt/registry/public-namespaces/serialt/providers

选中对应的provider，`Resync provider`



### 六、Registry Mirror

terraform 的 provider 托管在海外，使用时候会经常因为网络问题导致下载失败，terraform 官方有 Provider Network Mirror Protocol，支持使用本地文件和 Static Website，制作方法都依赖于`terraform providers mirror `命令获取到的资源。

#### 1、制作离线 provider 资源

##### 在当前目录下生成 provider.tf 文件

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

##### 创建存储的目录，并下载 provider

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

#### 2、本地文件镜像

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

#### 3、网络镜像

##### nginx配置文件

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

##### 上传下载的 provider 资源到 web 根目录

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



#### 4、基于脚本下载 provider 离线资源

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

