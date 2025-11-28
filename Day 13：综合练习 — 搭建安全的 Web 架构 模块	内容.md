Day 13：综合练习 — 搭建安全的 Web 架构
模块	内容
主题	综合练习
学习目标	整合网络、存储、计算、安全治理与监控知识，搭建一个高可用、安全的 Web 应用架构。
实战任务	构建包含前端、后端、数据库与存储的 Web 系统，配置安全组、私有访问、监控告警，实现从部署到治理的完整闭环。
🧠 一、总体架构设计

我们要搭建的架构是一个典型的企业级安全 Web 环境：

         Internet
             │
      ┌──────┴────────┐
      │  Azure Firewall│
      └──────┬────────┘
             │
       Hub VNet (10.0.0.0/16)
        ├── NSG (in/out rules)
        ├── Subnet-App   (Front-end VM)
        ├── Subnet-DB    (Private SQL)
        └── Subnet-Storage (Private Endpoint)
             │
       Private Endpoint → Azure Storage
             │
       Log Analytics + Azure Monitor
             │
        Policy + Cost Management


目标：

前端 Web VM 可安全访问；

后端数据库仅限内部访问；

存储账户通过 Private Endpoint 私有访问；

整体架构受 NSG、Policy、Monitor、Firewall 保护；

监控仪表盘展示运行状态。

⚙️ 二、实战任务步骤
任务 1：创建资源组与虚拟网络
az group create --name secure-web-rg --location eastus

az network vnet create \
  --resource-group secure-web-rg \
  --name secure-vnet \
  --address-prefix 10.0.0.0/16 \
  --subnet-name subnet-app \
  --subnet-prefix 10.0.1.0/24


创建数据库子网：

az network vnet subnet create \
  --resource-group secure-web-rg \
  --vnet-name secure-vnet \
  --name subnet-db \
  --address-prefix 10.0.2.0/24

任务 2：创建 NSG（网络安全组）并配置规则
az network nsg create \
  --resource-group secure-web-rg \
  --name web-nsg

# 允许 HTTP (80)
az network nsg rule create \
  --resource-group secure-web-rg \
  --nsg-name web-nsg \
  --name AllowHTTP \
  --protocol Tcp \
  --direction Inbound \
  --priority 100 \
  --destination-port-ranges 80 \
  --access Allow

# 允许 SSH (22)
az network nsg rule create \
  --resource-group secure-web-rg \
  --nsg-name web-nsg \
  --name AllowSSH \
  --protocol Tcp \
  --direction Inbound \
  --priority 110 \
  --destination-port-ranges 22 \
  --access Allow


将 NSG 关联到 App 子网：

az network vnet subnet update \
  --resource-group secure-web-rg \
  --vnet-name secure-vnet \
  --name subnet-app \
  --network-security-group web-nsg

任务 3：创建虚拟机作为 Web Server
az vm create \
  --resource-group secure-web-rg \
  --name web-vm \
  --image Ubuntu2204 \
  --size Standard_B1s \
  --vnet-name secure-vnet \
  --subnet subnet-app \
  --admin-username azureuser \
  --generate-ssh-keys


安装 Nginx：

az vm run-command invoke \
  --resource-group secure-web-rg \
  --name web-vm \
  --command-id RunShellScript \
  --scripts "sudo apt update && sudo apt install nginx -y && echo '<h1>Azure Secure Web</h1>' | sudo tee /var/www/html/index.html"

任务 4：创建数据库（可选：Azure SQL / MySQL）

创建 Azure SQL Server 与数据库：

az sql server create \
  --name mysecuresql$RANDOM \
  --resource-group secure-web-rg \
  --location eastus \
  --admin-user sqladmin \
  --admin-password "StrongP@ssword123"

az sql db create \
  --resource-group secure-web-rg \
  --server mysecuresql123 \
  --name webdb \
  --service-objective S0


禁用公共访问，仅允许 VNet 访问：

az sql server firewall-rule delete \
  --name AllowAllWindowsAzureIps \
  --server mysecuresql123 \
  --resource-group secure-web-rg


配置 Private Endpoint：

az network private-endpoint create \
  --name pe-sql \
  --resource-group secure-web-rg \
  --vnet-name secure-vnet \
  --subnet subnet-db \
  --private-connection-resource-id $(az sql server show --name mysecuresql123 --resource-group secure-web-rg --query id -o tsv) \
  --group-id sqlServer \
  --connection-name pe-sql-conn

任务 5：创建 Storage 并配置 Private Endpoint
az storage account create \
  --name securestorage$RANDOM \
  --resource-group secure-web-rg \
  --location eastus \
  --sku Standard_LRS \
  --kind StorageV2


绑定 Private Endpoint：

az network private-endpoint create \
  --name pe-storage \
  --resource-group secure-web-rg \
  --vnet-name secure-vnet \
  --subnet subnet-db \
  --private-connection-resource-id $(az storage account show --name securestorage123 --query id -o tsv) \
  --group-id blob \
  --connection-name pe-storage-conn

任务 6：启用 Azure Monitor + Log Analytics

创建 Log Analytics 工作区：

az monitor log-analytics workspace create \
  --resource-group secure-web-rg \
  --workspace-name secure-web-ws \
  --location eastus


安装 VM 监控代理：

az vm extension set \
  --publisher Microsoft.EnterpriseCloud.Monitoring \
  --name OmsAgentForLinux \
  --resource-group secure-web-rg \
  --vm-name web-vm \
  --settings '{"workspaceId": "<workspace-id>"}' \
  --protected-settings '{"workspaceKey": "<workspace-key>"}'


在 Portal → Log Analytics → Logs 输入：

Heartbeat
| where Computer == "web-vm"


✅ 若返回结果，说明监控已生效。

任务 7：应用 Azure Policy 控制与成本治理

启用策略：必须有 environment 标签；

限制 VM SKU：仅 B1s、B2s；

设置每月预算（例如 $20）并邮件告警。

示例：

az consumption budget create \
  --name "web-budget" \
  --category cost \
  --amount 20 \
  --time-grain monthly \
  --start-date 2025-01-01 \
  --end-date 2025-12-31 \
  --notification '{"Operator": "GreaterThan", "Threshold": 80, "ContactEmails": ["your@mail.com"]}'

任务 8：启用基础防护与日志审计

启用 Defender for Cloud：

az security pricing create --name VirtualMachines --tier Standard


启用诊断日志输出到 Log Analytics：

az monitor diagnostic-settings create \
  --name vm-diagnostics \
  --resource $(az vm show --name web-vm --resource-group secure-web-rg --query id -o tsv) \
  --workspace $(az monitor log-analytics workspace show --name secure-web-ws --resource-group secure-web-rg --query id -o tsv)

任务 9：创建仪表盘展示系统健康

进入 Portal → Dashboard → “New Dashboard”
添加以下模块：

VM CPU / 内存实时图表；

Storage 读写指标；

SQL 连接延迟；

网络流量；

安全告警列表。

保存为：

Secure-Web-Infra-Dashboard

✅ 三、验证成果

完成后，你将拥有：

可访问的 Web 服务（Nginx）；

安全的数据库连接（Private Endpoint）；

封闭的内部网络结构（NSG + Private DNS）；

成本受控（预算 + Policy）；

可观测性（Monitor + Dashboard）；

安全防护（Defender for Cloud）。

💡 可通过浏览器访问：

http://<web-vm-public-ip>


你将看到：

<h1>Azure Secure Web</h1>

🧠 四、延伸与思考

安全与合规

不暴露数据库公网；

开启 NSG 最小化规则；

禁止匿名存储访问；

启用 Defender for Cloud。

可扩展性

前端可替换为 App Service；

数据层可迁移至 Azure SQL MI；

通过 Azure Load Balancer 实现高可用。

自动化部署

下一阶段可将此架构转化为 ARM / Bicep / Terraform 模板；

在 CI/CD 中自动化创建与销毁环境。

📘 今日总结

你完成了第一个完整的 Azure 企业级 Web 架构实战；

整合了网络（VNet、NSG）、计算（VM）、存储（Blob）、治理（Policy）、监控（Monitor）；

理解了安全、合规与成本的统一治理；

构建了一个可持续、可监控、可防御的云端系统。