+++

title = 'Gitlab'
date = 2023-12-04T20:26:27+08:00
draft = false

tags = ["gitlab","git"]
categories = ["DevOps"]

+++

# Dev terraform provider

基于新框架tpf开发

开发示例：

* https://github.com/hashicorp/terraform-provider-scaffolding-framework

* https://github.com/serialt/terraform-provider-demo



main.go

```go
func main() {
    var debug bool

    flag.BoolVar(&debug, "debug", false, "debug terraform provider")
    flag.Parse()

    opts := providerserver.ServeOpts{

        Address: "hashicorp.com/edu/hashicups",
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

    "github.com/hashicorp/terraform-plugin-framework/datasource"
    "github.com/hashicorp/terraform-plugin-framework/provider"
    "github.com/hashicorp/terraform-plugin-framework/provider/schema"
    "github.com/hashicorp/terraform-plugin-framework/resource"
)

var (
    _ provider.Provider = &hashicupsProvider{}
)

func New(version string) func() provider.Provider {
    return func() provider.Provider {
        return &hashicupsProvider{
            version: version,
        }
    }
}


type hashicupsProvider struct {
    version string
}

// Metadata returns the provider type name.
func (p *hashicupsProvider) Metadata(_ context.Context, _ provider.MetadataRequest, resp *provider.MetadataResponse) {
    resp.TypeName = "hashicups"
    resp.Version = p.version
}

// Schema defines the provider-level schema for configuration data.
func (p *hashicupsProvider) Schema(_ context.Context, _ provider.SchemaRequest, resp *provider.SchemaResponse) {
    resp.Schema = schema.Schema{}
}

// Configure prepares a HashiCups API client for data sources and resources.
func (p *hashicupsProvider) Configure(ctx context.Context, req provider.ConfigureRequest, resp *provider.ConfigureResponse) {
}

// DataSources defines the data sources implemented in the provider.
func (p *hashicupsProvider) DataSources(_ context.Context) []func() datasource.DataSource {
    return nil
}

// Resources defines the resources implemented in the provider.
func (p *hashicupsProvider) Resources(_ context.Context) []func() resource.Resource {
    return nil
}

```

data_source.go

```go
package provider

import (
  "context"

  "github.com/hashicorp/terraform-plugin-framework/datasource"
  "github.com/hashicorp/terraform-plugin-framework/datasource/schema"
)

// Ensure the implementation satisfies the expected interfaces.
var (
  _ datasource.DataSource = &coffeesDataSource{}
)

// NewCoffeesDataSource is a helper function to simplify the provider implementation.
func NewCoffeesDataSource() datasource.DataSource {
  return &coffeesDataSource{}
}

// coffeesDataSource is the data source implementation.
type coffeesDataSource struct{}

// Metadata returns the data source type name.
func (d *coffeesDataSource) Metadata(_ context.Context, req datasource.MetadataRequest, resp *datasource.MetadataResponse) {
  resp.TypeName = req.ProviderTypeName + "_coffees"
}

// Schema defines the schema for the data source.
func (d *coffeesDataSource) Schema(_ context.Context, _ datasource.SchemaRequest, resp *datasource.SchemaResponse) {
  resp.Schema = schema.Schema{}
}

// Read refreshes the Terraform state with the latest data.
func (d *coffeesDataSource) Read(ctx context.Context, req datasource.ReadRequest, resp *datasource.ReadResponse) {
}

```

