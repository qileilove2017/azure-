Day 7：复盘与综合练习（完整基础架构 Demo）
模块	内容
主题	复盘与综合实践：构建可一键部署的 Azure 基础架构
学习目标	巩固前 6 天知识，掌握自动化部署、网络互联、安全配置的整体思路。
实战任务	使用 Azure CLI 构建一个完整的云环境，包括 RG、VNet、NSG、Storage、VM、RBAC 权限。
🧠 一、知识总览回顾

在前 6 天，你已经学会了以下关键能力：

模块	内容	CLI 示例
Day1	云计算模型与 Azure 概览	az login, az group create
Day2	Portal / CLI / PowerShell 操作	az vm list, az resource list
Day3	Subscription / Resource Group / Region	az account list, az group list
Day4	Virtual Machine 与网络	az vm create, az network vnet create
Day5	Storage (Blob / File / Table)	az storage blob upload
Day6	RBAC / IAM / PIM 权限管理	az role assignment create

今天你将把这些模块“拼起来”，形成一个端到端的基础设施。

⚙️ 二、架构目标

你将创建如下架构（适合后续用于 Web、API 或测试环境）：

```
    learn-azure-rg (资源组)
    │
    ├── demo-vnet (虚拟网络 10.0.0.0/16)
    │   ├── frontend-subnet (10.0.1.0/24)
    │   └── backend-subnet (10.0.2.0/24)
    │
    ├── demo-nsg (网络安全组)
    │   ├── Allow-HTTP (TCP 80)
    │   ├── Allow-SSH  (TCP 22)
    │
    ├── mystorageXXX (存储账户)
    │   ├── Blob 容器：webfiles
    │
    └── demo-vm (Ubuntu VM)
        ├── 网络接口绑定 frontend-subnet
        ├── 公网 IP
        └── RBAC 授权 “Contributor” 给指定用户
```

🧰 三、实战任务
任务 1：创建资源组
az group create \
  --name learn-azure-rg \
  --location eastus

任务 2：创建虚拟网络和子网
az network vnet create \
  --resource-group learn-azure-rg \
  --name demo-vnet \
  --address-prefix 10.0.0.0/16 \
  --subnet-name frontend-subnet \
  --subnet-prefix 10.0.1.0/24

az network vnet subnet create \
  --resource-group learn-azure-rg \
  --vnet-name demo-vnet \
  --name backend-subnet \
  --address-prefix 10.0.2.0/24

任务 3：创建网络安全组并添加规则
az network nsg create \
  --resource-group learn-azure-rg \
  --name demo-nsg

# 允许 SSH
az network nsg rule create \
  --resource-group learn-azure-rg \
  --nsg-name demo-nsg \
  --name AllowSSH \
  --protocol Tcp --direction Inbound \
  --priority 1000 --source-address-prefix '*' \
  --source-port-range '*' --destination-port-range 22 \
  --access Allow

# 允许 HTTP
az network nsg rule create \
  --resource-group learn-azure-rg \
  --nsg-name demo-nsg \
  --name AllowHTTP \
  --protocol Tcp --direction Inbound \
  --priority 1001 --destination-port-range 80 \
  --access Allow

任务 4：创建存储账户与容器
az storage account create \
  --name mystorage$RANDOM \
  --resource-group learn-azure-rg \
  --location eastus \
  --sku Standard_LRS \
  --kind StorageV2

az storage container create \
  --name webfiles \
  --account-name mystorage123 \
  --public-access blob


上传一个测试文件：

echo "<h1>Hello Azure!</h1>" > index.html
az storage blob upload \
  --account-name mystorage123 \
  --container-name webfiles \
  --name index.html \
  --file ./index.html

任务 5：创建 VM 并绑定网络组件
# 公网 IP
az network public-ip create \
  --resource-group learn-azure-rg \
  --name demo-ip

# 网卡
az network nic create \
  --resource-group learn-azure-rg \
  --name demo-nic \
  --vnet-name demo-vnet \
  --subnet frontend-subnet \
  --network-security-group demo-nsg \
  --public-ip-address demo-ip

# 创建 VM
az vm create \
  --resource-group learn-azure-rg \
  --name demo-vm \
  --image Ubuntu2204 \
  --admin-username azureuser \
  --generate-ssh-keys \
  --nics demo-nic \
  --size Standard_B1s

任务 6：配置 RBAC 权限

假设你有一个团队成员邮箱 devuser@yourtenant.onmicrosoft.com，为他授予 Contributor 权限：

az role assignment create \
  --assignee devuser@yourtenant.onmicrosoft.com \
  --role "Contributor" \
  --resource-group learn-azure-rg


验证：

az role assignment list \
  --resource-group learn-azure-rg \
  --output table

任务 7：验证 VM 网络与 Web 服务

SSH 登录虚拟机：

az vm show -d -g learn-azure-rg -n demo-vm --query publicIps -o tsv
ssh azureuser@<public-ip>


安装 Nginx：

sudo apt update
sudo apt install nginx -y
sudo systemctl start nginx
sudo systemctl enable nginx


访问浏览器：

http://<your-public-ip>/


你应看到 “Welcome to Nginx!” 页面。

任务 8：（可选）部署 HTML 页面
sudo rm /var/www/html/index.nginx-debian.html
sudo wget https://mystorage123.blob.core.windows.net/webfiles/index.html -O /var/www/html/index.html


刷新浏览器，你应看到你上传的 <h1>Hello Azure!</h1> 页面。

任务 9：清理资源
az group delete --name learn-azure-rg --yes --no-wait

✅ 四、验证成果

完成后你应具备：

一个完整的 Azure 基础架构环境；

可 SSH 登录的虚拟机；

公网访问的 Web 服务；

存储 Blob 文件可供下载；

RBAC 权限控制；

网络安全组(NSG)保护机制。

🧠 五、延伸与思考

模块化思维

你可以把以上 CLI 命令整理为一个 Shell 脚本，实现“一键部署”。

之后在 Day 22 学 Terraform 时，你会把这些逻辑转为 IaC（基础设施即代码）。

安全治理建议

不要长期暴露 22 端口；

使用 Azure Bastion 登录；

开启成本告警（Cost Management）。

架构扩展方向

加入数据库层（Azure SQL / Cosmos DB）；

添加 Application Gateway + WAF；

将 Web 层容器化（AKS）。

📘 今日总结

你完成了从零搭建 Azure 云环境的全过程；

巩固了资源、网络、计算、存储、安全、身份六大核心模块；

掌握了基础架构组合逻辑与一键部署方法；

打下了进入 DevOps + Terraform + AI 服务实战阶段（Week 2–8） 的坚实基础。