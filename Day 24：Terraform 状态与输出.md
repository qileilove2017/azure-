Day 24：Terraform 状态与输出

主题：Terraform 状态与输出｜学习目标：理解 state 文件与远程存储｜实战任务：使用 Azure Storage 远程状态

1) 你必须搞懂的：Terraform state 到底是什么？

Terraform 每次执行 apply 后，会生成一个 state 文件（默认本地 terraform.tfstate），它的意义不是“记录日志”，而是：

记录 Terraform 管理的资源“真实存在的 ID/属性”

用于对比“当前状态 vs 期望状态”来生成 plan

用于跨资源引用（例如 output、依赖关系）

如果你删了 state，Terraform 会“失忆”，可能导致：

资源重复创建

无法正确更新/删除

团队协作直接崩溃

所以：state 是 Terraform 的灵魂。

2) 为什么必须用 Remote State（远程状态）？

本地 state 只适合个人 demo。团队/企业一定要 remote state，原因很现实：

多人协作：大家共享同一份 state

锁定（Lock）：避免两个人同时 apply 把资源搞乱（Azure Blob 支持锁/一致性） 
opentofu.org

安全：state 可能包含敏感信息（比如某些资源属性），不能乱发

可审计：可配合权限体系（RBAC）

推荐远程存储：Azure Storage Blob Container（最常见、最成熟） 
Microsoft Learn

3) 远程 state 的“最佳做法”优先级

你会看到两种配置方式：

A. 使用 Storage Account Access Key（简单但不够安全）

需要在 backend 写 access_key。Microsoft Learn 示例也是这样入门 
Microsoft Learn

缺点：你得管理 Key，且很多企业会禁用 Shared Key。

B. 使用 Microsoft Entra ID（AAD）认证（更企业级，推荐）

Terraform azurerm backend 支持 use_azuread_auth=true 等方式，让你用 Entra ID（RBAC）访问 Storage 数据平面 
HashiCorp Developer

优点：更符合企业安全策略（尤其是“禁用存储 Key”时非常关键）。

今天我会把 两种都教你：先用 Access Key 跑通，再给你 AAD/RBAC 的升级方案（你以后在公司更可能用它）。

✅ 实战 Part A：用 Azure Storage 建 Remote State（Access Key 方式）
Step A1：创建 RG + Storage Account + Container

用 Azure CLI（你前面已经用过）：

# 1) 变量（按需改）
RG=tfstate-rg
LOC=eastus
SA=tfstate$RANDOM
CONTAINER=tfstate

# 2) 资源组
az group create -n $RG -l $LOC

# 3) Storage Account（建议用 Standard_LRS）
az storage account create \
  -n $SA \
  -g $RG \
  -l $LOC \
  --sku Standard_LRS \
  --kind StorageV2

# 4) 获取 Storage Key
ACCOUNT_KEY=$(az storage account keys list -g $RG -n $SA --query "[0].value" -o tsv)

# 5) 创建 Blob Container
az storage container create \
  --name $CONTAINER \
  --account-name $SA \
  --account-key $ACCOUNT_KEY


这一步完成后，你已经具备远程 state 的“存储容器”。

Microsoft Learn 的流程也是：先建 storage account/container，再配置 backend 
Microsoft Learn

Step A2：在 Terraform 里配置 backend（关键点：backend 只能在 init 时生效）

在你的 envs/dev/provider.tf（或单独 backend.tf）加入：

terraform {
  backend "azurerm" {
    resource_group_name  = "tfstate-rg"
    storage_account_name = "tfstate12345"   # 换成你的 $SA
    container_name       = "tfstate"
    key                  = "dev.terraform.tfstate"
    access_key           = "xxxxxxxxxxxxx"  # 换成你的 $ACCOUNT_KEY（不要提交到 Git）
  }
}


⚠️注意两点：

access_key 不要提交 Git（建议用环境变量或 backend config 文件）

修改 backend 后要重新 terraform init -reconfigure

Step A3：更安全的写法（把 key 放到 init 参数里，不写入 tf 文件）

创建一个文件：backend-dev.hcl（不提交到 Git，或放到安全渠道）：

resource_group_name  = "tfstate-rg"
storage_account_name = "tfstate12345"
container_name       = "tfstate"
key                  = "dev.terraform.tfstate"
access_key           = "xxxxxxxxxxxxx"


然后运行：

terraform init -reconfigure -backend-config=backend-dev.hcl
terraform plan
terraform apply


这样你的 Terraform 代码仓库里不会出现 access_key 明文。

Step A4：验证 remote state 是否生效

去 Azure Portal 或用 CLI 看 Blob：

az storage blob list \
  --container-name tfstate \
  --account-name $SA \
  --account-key $ACCOUNT_KEY \
  -o table


你会看到类似：

dev.terraform.tfstate

说明 state 已经进 Blob 了。

✅ 实战 Part B：企业推荐方式（AAD / Entra ID + RBAC，无需存储 Key）

Terraform 官方文档明确支持：use_azuread_auth = true（也可用环境变量 ARM_USE_AZUREAD） 
HashiCorp Developer

Step B1：给你的身份赋权（Storage 数据平面权限）

在 Storage Account 的 Access control (IAM) 里给当前用户 / Service Principal 分配角色（至少一个）：

Storage Blob Data Contributor（最常用）

或更高：Storage Blob Data Owner

这是访问 Blob 数据平面的权限，不是管理平面 Reader/Contributor。

Step B2：backend 配置改为 AAD 模式

在 backend 中加：

terraform {
  backend "azurerm" {
    resource_group_name  = "tfstate-rg"
    storage_account_name = "tfstate12345"
    container_name       = "tfstate"
    key                  = "dev.terraform.tfstate"

    use_azuread_auth     = true
  }
}


然后你需要一个已登录的 Azure CLI 会话：

az login
terraform init -reconfigure


use_azuread_auth 的含义与相关环境变量，Terraform 官方写得很清楚 
HashiCorp Developer

🧠 4) State 文件里到底包含什么？有什么“坑”？
你必须记住的 3 个现实问题

state 可能包含敏感信息（某些资源属性、output），所以要控制访问权限。

不要手动编辑 state（除非你非常清楚后果）。

state 迁移要用 terraform 自带的 init / state 命令，不要自己复制粘贴文件。

📤 5) 输出（outputs）在 Remote State 中怎么用？

你 Day 23 写了很多 outputs（如 subnet_id、vm_private_ip）。当你用 remote state 后：

outputs 仍然会写入 state

其他 Terraform 项目可以通过 terraform_remote_state 读取（后面 Day 25/26 很常用）

示例（给你一个概念，今天不强制做）：

data "terraform_remote_state" "network" {
  backend = "azurerm"
  config = {
    resource_group_name  = "tfstate-rg"
    storage_account_name = "tfstate12345"
    container_name       = "tfstate"
    key                  = "dev.terraform.tfstate"
    use_azuread_auth     = true
  }
}

output "subnet_id_from_remote" {
  value = data.terraform_remote_state.network.outputs.subnet_id
}

✅ Day 24 验收标准（你今天必须完成）

完成后你应该能做到：

能解释 state 的作用（对比、依赖、资源跟踪）

能解释 remote state 为什么必须（协作、锁、审计）

用 Azure Storage Blob Container 存远程 state 成功 
Microsoft Learn

会用 terraform init -reconfigure 切换 backend

（加分）能用 AAD/RBAC 模式，不依赖 access key 
HashiCorp Developer