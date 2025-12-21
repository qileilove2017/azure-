Day 23：Terraform 模块化

主题：Terraform 模块化｜学习目标：编写模块与变量文件｜实战任务：创建 RG、VNet、VM 模块

🧠 一、为什么要模块化？

如果你把 RG、VNet、VM 全写在一个 main.tf 里，很快会遇到：

代码重复（dev/prod 两套复制粘贴）

参数散落（命名、CIDR、SKU 到处是）

维护困难（一个改动影响全部）

难以复用（另一个项目想复用只能拷文件）

Terraform 模块化要解决的就是：把基础设施做成“组件”。

你今天学完之后，你的 IaC 会变成这种工程化结构：

infra/
  envs/
    dev/
    prod/
  modules/
    rg/
    vnet/
    vm/

🧠 二、模块化的核心概念（必须理解）
1) Module = 一组 Terraform 资源的封装

模块内部通常包含：

main.tf：资源定义

variables.tf：输入参数

outputs.tf：输出给外部引用的数据

2) Root Module vs Child Module

Root module：你在 envs/dev 里写的入口

Child module：modules/rg、modules/vnet 这种可复用组件

3) Input / Output

模块之间靠 输入变量 与 输出值 连接起来：

RG 模块输出 resource_group_name

VNet 模块输入 RG 名称

VM 模块输入 subnet id、rg 名称

📁 三、推荐目录结构（今天直接按这个建）

在你的 repo 里创建：

terraform-iac/
  modules/
    rg/
      main.tf
      variables.tf
      outputs.tf
    vnet/
      main.tf
      variables.tf
      outputs.tf
    vm/
      main.tf
      variables.tf
      outputs.tf
  envs/
    dev/
      main.tf
      provider.tf
      variables.tf
      terraform.tfvars
      outputs.tf


今天只做 dev 环境，prod 你后面复制一份 envs/prod 即可。

🧱 四、模块 1：RG 模块（modules/rg）
modules/rg/variables.tf
variable "name" {
  type        = string
  description = "Resource group name"
}

variable "location" {
  type        = string
  description = "Azure region"
}

modules/rg/main.tf
resource "azurerm_resource_group" "this" {
  name     = var.name
  location = var.location
}

modules/rg/outputs.tf
output "name" {
  value = azurerm_resource_group.this.name
}

output "location" {
  value = azurerm_resource_group.this.location
}

🌐 五、模块 2：VNet 模块（modules/vnet）
modules/vnet/variables.tf
variable "name" { type = string }
variable "location" { type = string }
variable "resource_group_name" { type = string }

variable "address_space" {
  type = list(string)
}

variable "subnet_name" { type = string }

variable "subnet_prefixes" {
  type = list(string)
}

modules/vnet/main.tf
resource "azurerm_virtual_network" "this" {
  name                = var.name
  location            = var.location
  resource_group_name = var.resource_group_name
  address_space       = var.address_space
}

resource "azurerm_subnet" "this" {
  name                 = var.subnet_name
  resource_group_name  = var.resource_group_name
  virtual_network_name = azurerm_virtual_network.this.name
  address_prefixes     = var.subnet_prefixes
}

modules/vnet/outputs.tf
output "vnet_id" {
  value = azurerm_virtual_network.this.id
}

output "subnet_id" {
  value = azurerm_subnet.this.id
}

🖥️ 六、模块 3：VM 模块（modules/vm）

今天我们用最小可跑通的 Linux VM（SSH Key）：

modules/vm/variables.tf
variable "name" { type = string }
variable "location" { type = string }
variable "resource_group_name" { type = string }

variable "subnet_id" { type = string }

variable "admin_username" {
  type    = string
  default = "azureuser"
}

variable "ssh_public_key" {
  type        = string
  description = "SSH public key content"
}

variable "vm_size" {
  type    = string
  default = "Standard_B1s"
}

modules/vm/main.tf
resource "azurerm_network_interface" "this" {
  name                = "${var.name}-nic"
  location            = var.location
  resource_group_name = var.resource_group_name

  ip_configuration {
    name                          = "ipconfig1"
    subnet_id                     = var.subnet_id
    private_ip_address_allocation = "Dynamic"
  }
}

resource "azurerm_linux_virtual_machine" "this" {
  name                = var.name
  location            = var.location
  resource_group_name = var.resource_group_name
  size                = var.vm_size
  admin_username      = var.admin_username

  network_interface_ids = [
    azurerm_network_interface.this.id
  ]

  admin_ssh_key {
    username   = var.admin_username
    public_key = var.ssh_public_key
  }

  os_disk {
    caching              = "ReadWrite"
    storage_account_type = "Standard_LRS"
  }

  source_image_reference {
    publisher = "Canonical"
    offer     = "0001-com-ubuntu-server-jammy"
    sku       = "22_04-lts"
    version   = "latest"
  }
}

modules/vm/outputs.tf
output "vm_id" {
  value = azurerm_linux_virtual_machine.this.id
}

output "private_ip" {
  value = azurerm_network_interface.this.ip_configuration[0].private_ip_address
}

🧩 七、Root Module（envs/dev）把三个模块串起来
envs/dev/provider.tf
terraform {
  required_version = ">= 1.5.0"
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
  }
}

provider "azurerm" {
  features {}
}

envs/dev/variables.tf
variable "location" { type = string }
variable "rg_name" { type = string }

variable "vnet_name" { type = string }
variable "address_space" { type = list(string) }
variable "subnet_name" { type = string }
variable "subnet_prefixes" { type = list(string) }

variable "vm_name" { type = string }
variable "admin_username" { type = string }
variable "ssh_public_key" { type = string }
variable "vm_size" { type = string }

envs/dev/terraform.tfvars（变量文件）
location        = "eastus"
rg_name         = "rg-dev-iac"

vnet_name       = "vnet-dev"
address_space   = ["10.10.0.0/16"]
subnet_name     = "subnet-app"
subnet_prefixes = ["10.10.1.0/24"]

vm_name         = "vm-dev-01"
admin_username  = "azureuser"
ssh_public_key  = "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQ..."  # 替换为你的公钥
vm_size         = "Standard_B1s"

envs/dev/main.tf
module "rg" {
  source   = "../../modules/rg"
  name     = var.rg_name
  location = var.location
}

module "vnet" {
  source              = "../../modules/vnet"
  name                = var.vnet_name
  location            = module.rg.location
  resource_group_name = module.rg.name
  address_space       = var.address_space
  subnet_name         = var.subnet_name
  subnet_prefixes     = var.subnet_prefixes
}

module "vm" {
  source              = "../../modules/vm"
  name                = var.vm_name
  location            = module.rg.location
  resource_group_name = module.rg.name
  subnet_id           = module.vnet.subnet_id
  admin_username      = var.admin_username
  ssh_public_key      = var.ssh_public_key
  vm_size             = var.vm_size
}

envs/dev/outputs.tf
output "resource_group" {
  value = module.rg.name
}

output "subnet_id" {
  value = module.vnet.subnet_id
}

output "vm_private_ip" {
  value = module.vm.private_ip
}

▶️ 八、运行 Terraform（今天的验收）

进入 dev 环境目录运行：

cd terraform-iac/envs/dev
terraform init
terraform plan
terraform apply


看到创建资源：

RG：rg-dev-iac

VNet：vnet-dev

Subnet：subnet-app

VM：vm-dev-01

✅ 九、Day 23 验收标准（必须达成）

你今天完成后应做到：

✔ modules/rg、modules/vnet、modules/vm 都可独立复用
✔ envs/dev 用 tfvars 控制参数
✔ terraform plan 能明确显示新增资源
✔ terraform apply 成功创建 RG、VNet、Subnet、VM
✔ 输出能看到：resource_group、subnet_id、vm_private_ip

🧠 十、你今天最重要的收获（复盘）

今天你实现了三件非常工程化的事：

把基础设施拆成模块（可复用）

用 tfvars 实现“同代码多环境”

用模块 outputs 串联资源依赖（rg → vnet → vm）

这就是企业级 Terraform 的第一步。