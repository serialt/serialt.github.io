+++

title = 'Gitlab'
date = 2024-06-30T20:26:27+08:00
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

### 一、调试 provider

#### 1、debug terraform

```
# makefile
default: install

build:
        go build -v ./...

install: 
        go install -v ./...
```



vscode 调试 terraform provider

##### 1）vscode launch

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

##### 2）ready terraform code

##### 3）run terraform

```
TF_REATTACH_PROVIDERS='{"registry.terraform.io/serialt/message":{"Protocol":"grpc","ProtocolVersion":5,"Pid":50972,"Test":true,"Addr":{"Network":"unix","String":"/var/folders/vm/zlhwbdyj2031f088_w9q3f8w0000gn/T/plugin3117659601"}}}' terraform apply
```

dev模式覆盖

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



### 二、TPF框架

目前官方在维护的开发 provider sdk 有两个版本：

* terraform-plugin-sdk/v2
* terraform-plugin-framework

如果要支持老版本的terraform，建议是使用 terraform-plugin-sdk/v2 开发，否则还是建议使用terraform-plugin-framework开发。

TPF开发示例：https://github.com/hashicorp/terraform-provider-hashicups



main.go

```go
package main

import (
	"context"
	"flag"
	"log"

	"github.com/hashicorp/terraform-plugin-framework/providerserver"
	"github.com/serialt/terraform-provider-harbor/internal/provider"
)

//go:generate go run github.com/hashicorp/terraform-plugin-docs/cmd/tfplugindocs

func main() {
	var debug bool

	flag.BoolVar(&debug, "debug", false, "set to true to run the provider with support for debuggers like delve")
	flag.Parse()

	err := providerserver.Serve(context.Background(), provider.New, providerserver.ServeOpts{
		Address: "registry.terraform.io/serialt/harbor",
		Debug:   debug,
	})
	if err != nil {
		log.Fatal(err)
	}
}

```

provider.go

```go
// Copyright (c) HashiCorp, Inc.
// SPDX-License-Identifier: MPL-2.0

package provider

import (
	"context"
	"os"
	"strings"

	"github.com/goharbor/go-client/pkg/harbor"
	"github.com/hashicorp/terraform-plugin-framework/datasource"
	"github.com/hashicorp/terraform-plugin-framework/path"
	"github.com/hashicorp/terraform-plugin-framework/provider"
	"github.com/hashicorp/terraform-plugin-framework/provider/schema"
	"github.com/hashicorp/terraform-plugin-framework/resource"
	"github.com/hashicorp/terraform-plugin-framework/types"
	"github.com/hashicorp/terraform-plugin-log/tflog"
)

var _ provider.Provider = &HarborProvider{}

func New() provider.Provider {
	return &HarborProvider{}
}

type HarborProvider struct {
}

type HarborProviderModel struct {
	URL         types.String `tfsdk:"url"`
	Username    types.String `tfsdk:"username"`
	Password    types.String `tfsdk:"password"`
	BearerToken types.String `tfsdk:"bearer_token"`
	Insecure    types.Bool   `tfsdk:"insecure"`
}

func (p *HarborProvider) Metadata(ctx context.Context, req provider.MetadataRequest, resp *provider.MetadataResponse) {
	resp.TypeName = "harbor"
}

func (p *HarborProvider) Schema(_ context.Context, _ provider.SchemaRequest, resp *provider.SchemaResponse) {
	resp.Schema = schema.Schema{
		Description: "Harbor config",
		Attributes: map[string]schema.Attribute{
			"url": schema.StringAttribute{
				Description: "Harbor url.",
				Optional:    true,
			},
			"username": schema.StringAttribute{
				Description: "Harbor username.",
				Optional:    true,
			},
			"password": schema.StringAttribute{
				Description: "Harbor.",
				Optional:    true,
				Sensitive:   true,
			},
			"bearer_token": schema.StringAttribute{
				Description: "Harbor.",
				Optional:    true,
			},
			"insecure": schema.BoolAttribute{
				Description: "Harbor.",
				Optional:    true,
			},
		},
	}
}

func (p *HarborProvider) Configure(ctx context.Context, req provider.ConfigureRequest, resp *provider.ConfigureResponse) {
	tflog.Info(ctx, "Configuring harbor client")

	// Retrieve provider data from configuration
	var config HarborProviderModel
	diags := req.Config.Get(ctx, &config)
	resp.Diagnostics.Append(diags...)
	if resp.Diagnostics.HasError() {
		return
	}

	if config.URL.IsUnknown() {
		resp.Diagnostics.AddAttributeError(
			path.Root("url"),
			"Unknown url",
			"",
		)
	}
	if resp.Diagnostics.HasError() {
		return
	}
	url := os.Getenv("HARBOR_URL")
	username := os.Getenv("HARBOR_USERNAME")
	password := os.Getenv("HARBOR_PASSWORD")
	insecure := false
	if strings.HasSuffix(url, "/") {
		url = strings.Trim(url, "/")
	}
	if !config.URL.IsNull() {
		url = config.URL.ValueString()
	}
	if !config.Username.IsNull() {
		username = config.Username.ValueString()
	}
	if !config.Password.IsNull() {
		password = config.Password.ValueString()
	}
	if !config.Insecure.IsNull() {
		insecure = config.Insecure.ValueBool()
	}

	harborConfig := &harbor.ClientSetConfig{
		URL:      url,
		Insecure: insecure,
		Username: username,
		Password: password,
	}

	client, err := harbor.NewClientSet(harborConfig)

	if err != nil {
		resp.Diagnostics.AddError(
			"Unable to create harbor client",
			err.Error(),
		)
	}
	resp.DataSourceData = client
	resp.ResourceData = client
	tflog.Info(ctx, "Configured harbor client", map[string]any{"success": true})

}

func (p *HarborProvider) Resources(ctx context.Context) []func() resource.Resource {
	return []func() resource.Resource{
		NewEmailResource,
	}
}

func (p *HarborProvider) DataSources(ctx context.Context) []func() datasource.DataSource {
	return []func() datasource.DataSource{
		NewProjectDataSource,
	}
}

```

data_project.go

```go
package provider

import (
	"context"
	"fmt"
	"strconv"

	"github.com/goharbor/go-client/pkg/harbor"
	"github.com/goharbor/go-client/pkg/sdk/v2.0/client/project"
	"github.com/hashicorp/terraform-plugin-framework/datasource"
	"github.com/hashicorp/terraform-plugin-framework/datasource/schema"
	"github.com/hashicorp/terraform-plugin-framework/types"
)

// Ensure the implementation satisfies the expected interfaces.
var (
	_ datasource.DataSource              = &ProjectDataSource{}
	_ datasource.DataSourceWithConfigure = &ProjectDataSource{}
)

// NewProjectDataSource is a helper function to simplify the provider implementation.
func NewProjectDataSource() datasource.DataSource {
	return &ProjectDataSource{}
}

// ProjectDataSource is the data source implementation.
type ProjectDataSource struct {
	client *harbor.ClientSet
}

// ProjectDataSourceModel maps the data source schema data.
type ProjectDataSourceModel struct {
	ID                    types.Int64  `tfsdk:"id"`
	Name                  types.String `tfsdk:"name"`
	Type                  types.String `tfsdk:"type"`
	ProjectID             types.Int64  `tfsdk:"project_id"`
	Public                types.Bool   `tfsdk:"public"`
	VulnerabilityScanning types.Bool   `tfsdk:"vulnerability_scanning"`
}

// Metadata returns the data source type name.
func (d *ProjectDataSource) Metadata(_ context.Context, req datasource.MetadataRequest, resp *datasource.MetadataResponse) {
	resp.TypeName = req.ProviderTypeName + "_project"
}

// Schema defines the schema for the data source.
func (d *ProjectDataSource) Schema(_ context.Context, _ datasource.SchemaRequest, resp *datasource.SchemaResponse) {
	resp.Schema = schema.Schema{

		Description: "Fetches the data of project.",
		Attributes: map[string]schema.Attribute{
			"id": schema.Int64Attribute{
				Description: "Project id.",
				Optional:    true,
			},
			"name": schema.StringAttribute{
				Description: "Project name.",
				Optional:    true,
			},
			"type": schema.StringAttribute{
				Description: "Project type.",
				Optional:    true,
			},
			"project_id": schema.Int64Attribute{
				Description: "Project id.",
				Optional:    true,
			},
			"public": schema.BoolAttribute{
				Description: "Project type.",
				Optional:    true,
			},
			"vulnerability_scanning": schema.BoolAttribute{
				Description: "Project vulnerability scanning.",
				Optional:    true,
			},
		},
	}
}

// Read refreshes the Terraform state with the latest data.
func (d *ProjectDataSource) Read(ctx context.Context, req datasource.ReadRequest, resp *datasource.ReadResponse) {
	var state ProjectDataSourceModel

	resp.Diagnostics.Append(req.Config.Get(ctx, &state)...)
	if resp.Diagnostics.HasError() {
		return
	}

	var projectNameOrID string

	if !state.Name.IsNull() {
		projectNameOrID = state.Name.ValueString()
	} else if !state.ID.IsNull() {
		projectNameOrID = fmt.Sprint(state.ID.ValueInt64())
	} else if !state.ProjectID.IsNull() {
		projectNameOrID = fmt.Sprint(state.ProjectID.ValueInt64())
	} else {
		resp.Diagnostics.AddError(
			"Get project id or name failed from tf", "",
		)
		return
	}

	params := &project.GetProjectParams{
		ProjectNameOrID: projectNameOrID,
	}

	project, err := d.client.V2().Project.GetProject(ctx, params)
	if err != nil {
		resp.Diagnostics.AddError(
			"Get project msg from harbor failed",
			err.Error(),
		)
		return
	}
	public := getboolfromstring(project.Payload.Metadata.Public)
	var autoScan bool

	// if len(*project.Payload.Metadata.AutoScan) ==
	myAutoScan := project.Payload.Metadata.AutoScan
	if myAutoScan == nil {
		autoScan = false
	} else {
		autoScan = getboolfromstring(*project.Payload.Metadata.AutoScan)
	}

	var projectType string
	if project.Payload.RegistryID != 0 {
		projectType = "ProxyCache"

	} else {
		projectType = "Project"
	}

	newState := &ProjectDataSourceModel{
		ID:                    types.Int64Value(int64(project.Payload.ProjectID)),
		ProjectID:             types.Int64Value(int64(project.Payload.ProjectID)),
		Name:                  types.StringValue(project.Payload.Name),
		Public:                types.BoolValue(public),
		VulnerabilityScanning: types.BoolValue(autoScan),
		Type:                  types.StringValue(projectType),
	}
	// Set state
	diags := resp.State.Set(ctx, &newState)
	resp.Diagnostics.Append(diags...)
	if resp.Diagnostics.HasError() {
		return
	}
}

// Configure adds the provider configured client to the data source.
func (d *ProjectDataSource) Configure(_ context.Context, req datasource.ConfigureRequest, resp *datasource.ConfigureResponse) {
	if req.ProviderData == nil {
		return
	}

	client, ok := req.ProviderData.(*harbor.ClientSet)
	if !ok {
		resp.Diagnostics.AddError(
			"Unexpected Data Source Configure Type",
			fmt.Sprintf("Expected Client, got: %T. Please report this issue to the provider developers.", req.ProviderData),
		)

		return
	}

	d.client = client
}

func getboolfromstring(stringbool string) bool {
	boolbool, err := strconv.ParseBool(stringbool)
	if err != nil {
		return false
	}
	return boolbool
}

```







