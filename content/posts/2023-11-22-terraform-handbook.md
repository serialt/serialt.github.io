+++
title = 'Terraform handbook'
date = 2023-11-22T21:22:27+08:00
draft = false

tags = ["tf","terraform"]
categories = ["DevOps"]

+++

# Terraform handbook
参考链接：https://blog.gmem.cc/terraform



## 一、简介

Terraform用于实现基础设施即代码（infrastructure as code）—— 通过代码（配置文件）来描述基础设施的拓扑结构，并确保云上资源和此结构完全对应。Terraform有三个版本，我们主要关注Terraform CLI。

Terraform CLI主要包含以下组件：

* 命令行前端

* Terraform Language（以下简称TL，衍生自HashiCorp配置语言HCL）编写的、描述基础设施拓扑结构的配置文件。配置文件的组织方式是模块。本文使用术语“配置”（Configuration）来表示一整套描述基础设施的Terraform配置文件

* 针对各种云服务商的驱动（Provider），实现云资源的创建、更新和删除

云上资源不单单包括基础的IaaS资源，还可以是DNS条目、SaaS资源。事实上，通过开发Provider，你可以用Terraform管理任何资源。

Terraform会检查配置文件，并生成执行计划。计划描述了那些资源需要被创建、修改或删除，以及这些资源之间的依赖关系。Terraform会尽可能并行的对资源进行变更。当你更新了配置文件后，Terraform会生成增量的执行计划。



## 二、命令行

### 1、安装命令行

直接到https://www.terraform.io/downloads.html下载，存放到$PATH下即可。

### 2、基本特性

#### 1）切换工作目录

使用选项 -chdir=DIR

#### 2）Shell自动补全

使用 terraform -install-autocomplete安装自动完成脚本，使用 terraform -uninstall-autocomplete删除自动完成脚本。



### 3、资源地址

很多子命令接受资源地址参数，下面是一些例子：

```shell
# 资源类型.资源名
aws_instance.foo
# 资源类型.资源列表名[索引]
aws_instance.bar[1]
# 子模块foo的子模块bar中的
module.foo.module.bar.aws_instance.baz
```

### 4、配置文件

配置文件的路径可以通过环境变量 TF_CLI_CONFIG_FILE设置。非Windows系统中， $HOME/.terraformrc为默认配置文件路径。配置文件语法类似于TF文件：

```shell
# provider缓存目录
plugin_cache_dir   = "$HOME/.terraform.d/plugin-cache"
# 
disable_checkpoint = true
 
# 存放凭证信息，包括模块仓库、支持远程操作的系统的凭证
credentials "app.terraform.io" {
  token = "xxxxxx.atlasv1.zzzzzzzzzzzzz"
}
 
# 改变默认安装逻辑
provider_installation {
  # 为example.com提供本地文件系统镜像，这样安装example.com/*/*的provider时就不会去网络上请求
  # 默认路径是：
  # ~/.terraform.d/plugins/${host_name}/${namespace}/${type}/${version}/${target}
  # 例如：
  # ~/.terraform.d/plugins/hashicorp.com/edu/hashicups/0.3.1/linux_amd64/terraform-provider-hashicups_v0.3.1
  filesystem_mirror {
    path    = "/usr/share/terraform/providers"
    include = ["example.com/*/*"]
  }
  direct {
    exclude = ["example.com/*/*"]
  }
  
  # Terraform会在terraform init的时候，校验Provider的版本和checksum。Provider从Registry或者本地
  # 目录下载Provider。当我们开发Provider的时候，常常需要方便的测试临时Provider版本，这种Provider还
  # 没有关联版本号，也没有在Registry中注册Chencksum
  # 为了简化开发，可以配置dev_overrides，它能覆盖所有配置的安装方法
  dev_overrides {
    "hashicorp.com/edu/hashicups-pf" = "$(go env GOBIN)"
  }
}
```

### 5、init

配置工作目录，为使用其它命令做好准备。

Terraform命令需要在一个编写了Terraform配置文件的目录（配置根目录）下执行，它会在此目录下存储设置、缓存插件/模块，以及（默认使用Local后端时）存储状态数据。此目录必须进行初始化。

初始化后，会生成以下额外目录/文件：

.terraform目录，用于缓存provider和模块
如果使用Local后端，保存状态的terraform.tfstate文件。如果使用多工作区，则是terraform.tfstate.d目录。

对配置的某些变更，需要重新运行初始化，包括provider需求的变更、模块源/版本约束的变更、后端配置的变更。需要重新初始化时，其它命令可能会无法执行并提示你进行初始化。

命令 terraform get可以仅仅下载依赖的模块，而不执行其它init子任务。

运行 terraform init -upgrade会强制拉取最新的、匹配约束的版本并更新依赖锁文件。



### 6、validate

校验配置是否合法。



### 7、plan

显示执行计划，即当前配置将请求（结合state）哪些变更。Terraform的核心功能时创建、修改、删除基础设施对象，使基础设施的状态和当前配置匹配。当我们说运行Terraform时，主要是指plan/apply/destroy这几个命令。

terraform plan命令评估当前配置，确定其声明的所有资源的期望状态。然后比较此期望状态和真实基础设施的当前状态。它使用state来确定哪些真实基础设施对象和声明资源的对应关系，并且使用provider的API查询每个资源的当前状态。当确定到达期望状态需要执行哪些变更后，Terraform将其打印到控制台，它并不会执行任何实际的变更操作。

#### 计划模式

plan命令支持两种备选的工作模式：

* 销毁模式：创建一个计划，其目标是销毁所有当前存在于配置中的远程对象，留下一个空白的state。对应选项 -destroy

* 仅刷新模式：创建一个计划，其目标仅仅是更新state和根模块的输出值，以便和从Terraform之外对基础设施对象的变更匹配。对应选项 -refresh-only



#### 指定输入变量

使用选项 -**var** 'NAME=VALUE'可以指定输入变量，该选项可以使用多次。

使用选项 -**var**-file=FILENAME可以从文件读取输入变量，某些文件会自动读取



### 8、apply

应用执行计划，创建、更新设施对象。

apply会做plan的任何事情，并在其基础上，直接执行变更操作。默认情况下，apply即席的执行一次plan，你也可以直接使用已保存的plan

命令格式： terraform apply [options] [plan file]

#### 自动确认

选项 -auto-approve可以自动确认并执行所需操作，不需要人工确认。

#### 使用已有计划

如果指定plan file参数，则读取先前保存的计划并执行。

#### 计划模式

支持plan命令中关于计划模式的选项。





## 三、TF语言

### 1、块

配置文件由若干块（Block）组成，块的语法如下：

```shell
# Block header, which identifies a block
<BLOCK TYPE> "<BLOCK LABEL>" "<BLOCK LABEL>" "..." {
  # Block body
  <IDENTIFIER> = <EXPRESSION> # Argument
}
```

块是一个容器，它的作用取决于块的类型。块常常用来描述某个资源的配置。

取决于块的类型，标签的数量可以是0-N个。对于resource块，标签数量为两个。某些特殊的块，可能支持任意数量的标签。某些内嵌的块，例如network_interface，则不支持标签。

块体中可以包含若干参数（Argument），或者其它内嵌的块。参数用于将一个表达式分配到一个标识符，常常对应某个资源的一条属性。表达式可以是字面值，或者引用其它的值，正是这种引用让Terraform能够识别资源依赖关系。

直接位于配置文件最外层的块，叫做顶级块（Top-level Block），Terraform支持有限种类的顶级块。大部分Terraform特性，例如resource，基于顶级块实现。

下面是一个例子：

```shell
resource "aws_vpc" "main" {
  cidr_block = var.base_cidr_block
}

```



### 2、数据类型

| 类型       | 说明                                          |
| ---------- | --------------------------------------------- |
| string     | Unicode字符序列，基本形式 "hello"             |
| number     | 数字，形式 6.02                               |
| bool       | **true**或 **false**                          |
| list/tuple | 一系列的值，形式 ["us-west-1a", "us-west-1c"] |
| map/object | 键值对，形式 {name = "Mabel", age = 52}       |

### 3、空值

空值使用 **null**表示。



#### 4、字符串和模板

#### 转义字符

```shell
\n 换行
\r 回车
\t 制表
\" 引号
\\ 反斜杠
\uNNNN Unicode字符
\UNNNNNNNN Unicode字符
```

注意，在Heredoc中反斜杠不用于转义，可以使用：

$${ 字符串插值标记${
%%{ 模板指令标记%{

支持unix风格的字符串

```shell
block {
  value = <<EOT
hello
world
EOT
}

block {
  value = <<-EOT
  hello
    world
  EOT
}
```

要将对象转换为JSON或YAML，可以调用函数：

```shell
example = jsonencode({
    a = 1
    b = "hello"
})
```



### 5、操作符

逻辑操作符： ! && ||
算数操作符： * / % + -
比较操作符： >, >=, <, <= ==, !=

#### 条件表达式

```shell
condition ? true_val : false_val
 
var.a != "" ? var.a : "default-a"
```

#### for表达式

使用for表达式可以通过转换一种复杂类型输出，生成另一个复杂类型结果。输入中的每个元素，可以对应结果的0-1个元素。任何表达式可以用于转换，下面是使用upper函数将列表转换为大写：

```shell
[for s in var.list : upper(s)]
```

#### 输入类型

作为for表达式的输入的类型可以是list / set / tuple / map / object。可以为for声明两个临时符号，前一个表示index或key：

```shell
[for k, v in var.map : length(k) + length(v)]
```

#### 结果类型 

结果的类型取决于包围for表达式的定界符：

[] 表示生成的结果是元组
{} 表示生成的结果是object，你必须使用` =>`符号：` {for s in var.list : s => upper(s)}`

#### 输入过滤

包含一个可选的if子句可以对输入元素进行过滤：` [for s in var.list : upper(s) if s != ""]`

示例：

```shell
variable "users" {
  type = map(object({
    is_admin = boolean
  }))
}
 
locals {
  admin_users = {
    for name, user in var.users : name => user
    if user.is_admin
  }
}
```



#### splat表达式

splat表达式提供了更简单语法，在某些情况下代替for表达式：

```shell
[for o in var.list : o.id]
# 等价于
var.list[*].id
 
[for o in var.list : o.interfaces[0].name]
# 等价于
var.list[*].interfaces[0].name
```

#### 可选object属性

```shell
variable "with_optional_attribute" {
  type = object({
    a = string           # 必须属性
    b = optional(string) # 可选属性
  })
}

```

#### 版本约束

版本约束是一个特殊的字符串值，在引用module、使用provider时，或者通过terraform块的required_version时，需要用到版本约束：

```shell
# 版本范围区间
version = ">= 1.2.0, < 2.0.0"
 
# 操作符
=   等价于无操作符，限定特定版本
!=  排除特定版本
> >= < <= 限制版本范围
~>  允许最右侧的版本号片段的变化
```

#### depends_on

该元参数用于处理隐含的资源/模块依赖，这些依赖无法通过分析Terraform配置文件得到。从0.13版本开始，该元参数可用于模块。之前的版本仅仅用于资源。

depends_on的值是一个列表，其元素具必须是其它资源的引用，不支持任意表达式。

depends_on应当仅仅用作最后手段，避免滥用。

```shell
resource "aws_iam_role" "example" {
  name = "example"
  assume_role_policy = "..."
}
 
# 这个策略允许运行在EC2中的实例访问S3 API
resource "aws_iam_role_policy" "example" {
  name   = "example"
  role   = aws_iam_role.example.name
  policy = jsonencode({
    "Statement" = [{
      "Action" = "s3:*",
      "Effect" = "Allow",
    }],
  })
}
 
 
resource "aws_iam_instance_profile" "example" {
  # 这是可以自动分析出的依赖
  role = aws_iam_role.example.name
}
 
 
resource "aws_instance" "example" {
  ami           = "ami-a1b2c3d4"
  instance_type = "t2.micro"
 
  # 这是可以自动分析出的依赖，包括传递性依赖
  iam_instance_profile = aws_iam_instance_profile.example
 
  # 如果这个实例中的程序需要访问S3接口，我们需要用元参数显式的声明依赖
  # 从而分配策略
  depends_on = [
    aws_iam_role_policy.example,
  ]
} 
```

#### count

默认情况下，一个resource块代表单个云上基础设施对象。如果你想用一个resource块生成多个类似的资源，可以用count或for_each参数。

设置了此元参数的上下文中，可以访问名为 count的变量，它具有属性 index，为从0开始计数的资源实例索引。 示例：

```shell
resource "aws_instance" "server" {
  # 创建4个类似的实例
  count = 4
 
  ami           = "ami-a1b2c3d4"
  instance_type = "t2.micro"
 
  tags = {
    #              实例的索引作为tag的一部分
    Name = "Server ${count.index}"
  }
}
```

### for_each

如果资源的规格几乎完全一致，可以用count，否则，需要使用更加灵活的for_each元参数。

for_each的值必须是一个映射或set(string)，你可以在上下文中访问 **each**对象， 它具有 key和 value两个属性，如果for_each的值是集合，则key和value相等。示例：

```shell
resource "azurerm_resource_group" "rg" {
  for_each = {
    a_group = "eastus"
    another_group = "westus2"
  }
  # 对于每个键值对都会生成azurerm_resource_group资源
  name     = each.key
  location = each.value
}
 
 
resource "aws_iam_user" "the-accounts" {
  # 数组转换为集合
  for_each = toset( ["Todd", "James", "Alice", "Dottie"] )
  name     = each.key
}
```

```shell
variable "vpcs" {
  # 这里定义了map类型的变量，并且限定了map具有的键
  type = map(object({
    cidr_block = string
  }))
}
 
 
# 创建多个VPC资源
resource "aws_vpc" "example" {
  for_each = var.vpcs
  cidr_block = each.value.cidr_block
}
# 上述资源作为下面那个for_each的值
 
# 创建对应数量的网关资源
resource "aws_internet_gateway" "example" {
  # 为每个VPC创建一个网关
  #          资源作为值
  for_each = aws_vpc.example
 
  # 映射的值，在这里是完整的VPC对象
  vpc_id = each.value.id
}
 
 
# 输出所有VPC ID
output "vpc_ids" {
  value = {
    for k, v in aws_vpc.example : k => v.id
  }
 
  # 显式依赖网关资源，确保网关创建后，输出才可用
  depends_on = [aws_internet_gateway.example]
}
```

