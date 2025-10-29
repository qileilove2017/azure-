Day 3：Resource Group、Subscription、Region
模块	内容
主题	Resource Group、Subscription、Region
学习目标	理解 Azure 的逻辑组织层次（Management Group → Subscription → Resource Group → Resource），掌握订阅管理与区域选择原则。
实战任务	创建多个资源组（跨不同区域），探索订阅信息，并验证资源部署的地理差异。
🧠 一、核心知识讲解
1. Azure 资源组织结构（从上到下）
Management Group
   └── Subscription
         └── Resource Group
                └── Resource（VM、Storage、DB、Network…）

层级	作用	管理内容
Management Group	管理多个订阅	统一策略与访问控制
Subscription（订阅）	计费与资源隔离边界	成本、配额、访问权限
Resource Group（资源组）	逻辑容器	将一组资源组织在一起
Resource（资源）	实际服务实例	VM、Storage、Database 等

💡 要点理解：

订阅是计费单位；

资源组是逻辑组织单位；

同一资源组内的资源可位于不同区域，但建议保持一致；

删除资源组会连带删除所有资源。

2. Subscription（订阅）

每个订阅都有唯一 ID，与计费账户绑定。
订阅类型有：

Free Trial（免费试用）：赠送 $200 美元额度；

Pay-As-You-Go（按需付费）；

Visual Studio Subscription（开发者订阅）；

Enterprise Agreement（企业订阅）。

常用命令：

az account list --output table
az account set --subscription "<Subscription Name or ID>"

3. Region（区域）

Azure 在全球有 60+ 个区域（Region），每个区域包含多个数据中心。
示例：

区域	代号	位置
East US	eastus	弗吉尼亚，美国东部
West Europe	westeurope	荷兰
Southeast Asia	southeastasia	新加坡
Japan East	japaneast	东京
China East 2	chinaeast2	上海

💡 选择区域时的考虑：

靠近用户（降低延迟）；

符合法规（如中国区、欧洲 GDPR）；

可用的服务种类；

成本（部分区域价格略有差异）。

4. Resource Group 特性
特性	描述
生命周期一致	删除 RG 会删除内部资源
区域属性	RG 有自己的位置，但可以包含跨区资源
访问控制	可在 RG 级别分配 RBAC 权限
标签（Tags）	用于标识资源（例如环境、项目、成本中心）

示例 Tag：

--tags environment=dev project=demo costcenter=IT01

🧰 二、实战任务
任务 1：查看订阅信息
az account list --output table


输出类似：

Name                 CloudName    SubscriptionId                        IsDefault
-------------------  -----------  ------------------------------------  -----------
Azure subscription   AzureCloud   6df1a9b5-xxxx-xxxx-xxxx-45ce37d71dce  True

任务 2：创建跨区域资源组

创建美国东部资源组

az group create --name rg-eastus-demo --location eastus


创建日本东部资源组

az group create --name rg-japaneast-demo --location japaneast


创建东南亚（新加坡）资源组

az group create --name rg-southeast-demo --location southeastasia

任务 3：添加标签管理

为资源组添加标识标签：

az group update --name rg-eastus-demo --set tags.environment=dev tags.owner="yourname"


查看带标签的资源组：

az group list --query "[].{Name:name, Location:location, Tags:tags}" --output table

任务 4：验证区域差异

在不同区域的资源组中各创建一个 Storage Account：

az storage account create \
  --name eaststorage$RANDOM \
  --resource-group rg-eastus-demo \
  --location eastus \
  --sku Standard_LRS

az storage account create \
  --name japstorage$RANDOM \
  --resource-group rg-japaneast-demo \
  --location japaneast \
  --sku Standard_LRS


验证：

az storage account list --output table


Portal 上你会看到两个不同地理位置的存储账户。

任务 5：（可选）删除实验资源
az group delete --name rg-eastus-demo --yes --no-wait
az group delete --name rg-japaneast-demo --yes --no-wait
az group delete --name rg-southeast-demo --yes --no-wait

🧩 三、验证成果

✅ 你应当能做到：

理解 Subscription、Resource Group、Region 的层级关系；

使用命令列出订阅；

成功创建多个跨区域的资源组与资源；

为资源组添加标签并查询；

了解区域的差异与命名规范。

🧠 四、延伸与思考

资源规划练习
假设你要为企业构建：

生产环境（prod）

测试环境（test）
你会如何命名和组织 Resource Group？
示例：

RG-WebApp-Prod-EastUS
RG-WebApp-Test-EastUS


多订阅场景
在企业环境中，不同团队或项目往往使用不同 Subscription，你可以通过：

az account set --subscription "TeamA Subscription"


快速切换。

管理与治理建议

按环境（dev/test/prod）或系统功能划分 RG；

统一使用 Tag 管理成本与Owner；

定期清理无用资源组。

📘 今日总结

你掌握了 Azure 的核心组织结构（Management Group → Subscription → RG → Resource）；

理解了订阅是计费边界，RG 是逻辑组织单位；

学会了跨区域创建与管理资源；

了解了区域选择的重要性与标签的治理价值。