Day 2：Azure Portal、CLI、PowerShell 基础
模块	内容
主题	Azure Portal、CLI、PowerShell 基础
学习目标	掌握 Azure 资源的三种常用操作方式（Portal、CLI、PowerShell），理解命令行与 Portal 的区别与优势。
实战任务	使用 Azure CLI 和 PowerShell 创建虚拟机 (VM)、存储账户，并学会用命令查询、删除、和管理资源。
🧠 一、核心知识讲解
1. 三种管理方式的对比
工具	特点	适用场景
Azure Portal	图形化界面，操作直观，适合入门和临时配置	初学者 / 快速验证配置
Azure CLI	命令行方式，跨平台 (Windows/macOS/Linux)，支持脚本自动化	DevOps、持续部署
Azure PowerShell	深度集成 Windows 管理体系，支持对象化命令输出	系统管理员、混合云场景
2. Azure CLI 基本语法

Azure CLI 命令结构：

az <命令组> <操作> [参数]


例如：

az group create --name myGroup --location eastus


这里：

az group 表示资源组命令组

create 是创建操作

--name 和 --location 是参数

🧩 常用命令：

az login                      # 登录账户
az account show               # 查看当前订阅信息
az group list --output table  # 列出所有资源组
az resource list              # 列出所有资源

3. Azure PowerShell 基础命令

PowerShell 使用模块 Az，命令风格更接近面向对象的操作。

安装与登录

Install-Module -Name Az -Scope CurrentUser -Repository PSGallery -Force
Connect-AzAccount


常见操作

Get-AzSubscription
Get-AzResourceGroup
New-AzResourceGroup -Name "learn-powershell-rg" -Location "EastUS"

4. CLI 与 PowerShell 的区别（快速记忆）
维度	Azure CLI	PowerShell
平台	跨平台	主要用于 Windows
输出格式	JSON / Table	对象（可进一步处理）
脚本语言	Bash / Shell	PowerShell 脚本
优点	简洁、易学	强大、可处理复杂逻辑

💡 建议：CLI 用于跨平台自动化；PowerShell 用于复杂系统任务。

🧰 二、实战任务
任务 1：使用 Azure CLI 创建与查询资源

创建资源组

az group create --name cli-demo-rg --location eastus


创建存储账户

az storage account create \
  --name clistorage$RANDOM \
  --resource-group cli-demo-rg \
  --location eastus \
  --sku Standard_LRS


查询资源组中的所有资源

az resource list --resource-group cli-demo-rg --output table


删除资源组

az group delete --name cli-demo-rg --yes --no-wait

任务 2：使用 PowerShell 执行同样操作

登录 Azure

Connect-AzAccount


创建资源组

New-AzResourceGroup -Name "ps-demo-rg" -Location "EastUS"


创建存储账户

New-AzStorageAccount -ResourceGroupName "ps-demo-rg" `
  -Name "psstorage$((Get-Random).ToString())" `
  -Location "EastUS" `
  -SkuName "Standard_LRS" `
  -Kind "StorageV2"


列出资源

Get-AzResource -ResourceGroupName "ps-demo-rg"


删除资源组

Remove-AzResourceGroup -Name "ps-demo-rg" -Force

🧩 三、验证成果

✅ 你应当能够：

在 Portal 中看到 CLI/PowerShell 创建的资源组；

执行 az resource list 或 Get-AzResource 看到相同内容；

删除资源后，Portal 自动同步变化。

🧠 四、延伸与思考

输出格式差异实验
比较以下命令结果：

az group list --output json
az group list --output table


脚本自动化思考
你能否写一个 Shell 脚本或 PowerShell 脚本，自动创建 3 个资源组并打印它们的名称？

官方学习资料

Azure CLI 入门 (Microsoft Learn)

Azure PowerShell 文档

📘 今日总结

你掌握了三种 Azure 操作方式：Portal、CLI、PowerShell。

已能通过命令创建与管理资源。

理解了命令行自动化的优势。

熟悉了基本命令结构，为后续自动化部署打下基础。