Day 22：Terraform 基础（IaC）

主题：Terraform 基础｜学习目标：了解 IaC 概念｜实战任务：安装 Terraform + Provider 配置

🧠 一、什么是 IaC（Infrastructure as Code）？

IaC = 用代码定义基础设施，而不是手动点 Portal

传统方式 vs IaC：

方式	特点
手工点 Azure Portal	不可复现、易出错、无法审计
Shell 脚本	可执行但不可描述状态
Terraform（IaC）	声明式、可复现、可版本化

一句话总结 Terraform：

👉 你只需要“描述你想要的最终状态”，Terraform 负责把现实世界变成那个状态。

🔧 二、Terraform 在 DevOps 中的定位

在真实企业里，Terraform 通常处于这个位置：

Git Repo
   ↓
Terraform Code (.tf)
   ↓
terraform plan
   ↓
terraform apply
   ↓
Azure Resource (VNet / VM / Web App / Key Vault)


并且 Terraform 本身也会被 Azure DevOps Pipeline 调用（后面 Day 24+ 会做）。

🧠 三、Terraform 的三个核心概念（必须理解）
1️⃣ Provider（云厂商插件）

Provider 决定 Terraform 操作哪朵云、哪种资源。

例如：

Azure → azurerm

AWS → aws

GCP → google

2️⃣ Resource（资源）

资源是真正被创建的对象：

resource "azurerm_resource_group" "example" {
  name     = "rg-demo"
  location = "eastus"
}

3️⃣ State（状态）

Terraform 会维护一个 terraform.tfstate 文件：

记录 “我创建了什么”

用来对比 “现在 vs 目标”

是 Terraform 的灵魂

⚠️ State 不能随便删（后面会讲 remote state）。

⚙️ 四、实战任务 1：安装 Terraform
macOS（推荐，和你现在环境一致）
brew tap hashicorp/tap
brew install hashicorp/tap/terraform


验证：

terraform version


看到类似输出即成功：

Terraform v1.8.x

Windows（补充）

下载：https://developer.hashicorp.com/terraform/downloads

解压并加入 PATH

terraform version 验证

⚙️ 五、实战任务 2：准备 Azure 认证方式（非常重要）

Terraform 操作 Azure 必须有身份。

✅ 推荐方式（企业标准）：Azure CLI 登录

你已经在前面 Azure 学习中用过：

az login


确认当前订阅：

az account show


Terraform 会自动复用这个身份（非常方便）。

📁 六、实战任务 3：创建 Terraform 项目结构

在你的 DevOps / Infra Repo 中新建目录：

mkdir terraform-demo
cd terraform-demo


推荐最小结构：

terraform-demo/
 ├── main.tf
 ├── provider.tf
 ├── variables.tf
 └── outputs.tf


今天我们先写最基础的内容。

🧩 七、编写 Provider 配置（核心实战）
📄 provider.tf
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


解释（你要能讲清楚）：

required_providers：声明用哪个云插件

azurerm：Azure Resource Manager Provider

features {}：Azure Provider 的必填项

🧩 八、编写第一个资源（Resource Group）
📄 main.tf
resource "azurerm_resource_group" "demo" {
  name     = "rg-terraform-demo"
  location = "eastus"
}


这里有三个关键点：

resource "<类型>" "<逻辑名>" {
  属性 = 值
}


类型：azurerm_resource_group

逻辑名：demo

实际资源名：rg-terraform-demo

▶️ 九、运行 Terraform（第一次完整流程）
1️⃣ 初始化（下载 Provider）
terraform init


你会看到：

Terraform has been successfully initialized!

2️⃣ 预览变更（最重要的习惯）
terraform plan


你应该看到：

+ azurerm_resource_group.demo


👉 plan = 安全检查，不会真的创建资源

3️⃣ 应用变更（真正创建资源）
terraform apply


输入 yes 确认。

然后：

Terraform 创建 Resource Group

本地生成 terraform.tfstate

你可以去 Azure Portal 验证：
✔ Resource Group 已存在