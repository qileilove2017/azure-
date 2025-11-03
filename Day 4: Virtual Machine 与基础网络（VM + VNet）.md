Day 4：Virtual Machine 与基础网络（VM + VNet）
模块	内容
主题	Virtual Machine 与基础网络（VNet + NSG）
学习目标	掌握 Azure 虚拟机（VM）的创建方式、网络配置、SSH 登录与安全策略，理解虚拟网络（VNet）、子网（Subnet）、安全组（NSG）的基本概念。
实战任务	创建一个虚拟网络（VNet）和一台 Linux 虚拟机，通过 SSH 登录并验证连通性。
🧠 一、核心知识讲解
1. 什么是 Azure Virtual Machine (VM)

Azure VM 是 IaaS（基础设施即服务）的典型代表，提供云端虚拟计算资源。
你可以像在本地一样安装系统、运行服务、连接网络。

VM 的关键组成：

计算资源：CPU、内存、磁盘

网络配置：VNet、子网、NIC、Public IP

安全控制：NSG、身份认证（SSH Key / 密码）

存储类型：OS Disk + Data Disk

2. 什么是 Virtual Network (VNet)

VNet 是 Azure 的虚拟私有网络，用于隔离和连接资源。
它类似企业内部局域网，可定义子网、路由和安全策略。

VNet 层次结构：

VNet（虚拟网络）
 ├── Subnet（子网）
 │    ├── VM1
 │    ├── VM2
 │    └── Azure SQL
 └── NSG（Network Security Group）
      ├── 入站规则
      └── 出站规则

3. Network Security Group (NSG)

NSG 定义了入站 / 出站流量规则，类似防火墙。

类型	示例	说明
入站规则	允许 TCP 端口 22	SSH 登录
出站规则	允许所有流量	VM 访问外部资源
默认规则	阻止除已定义规则外的所有入站	安全性保证
4. SSH 密钥认证机制

相比密码登录，SSH 密钥更安全：

在本地生成一对密钥（public/private key）；

将公钥上传到 Azure；

登录时使用私钥认证。

生成命令（macOS/Linux）：

ssh-keygen -t rsa -b 2048

🧰 二、实战任务
任务 1：创建虚拟网络和子网
az network vnet create \
  --name demo-vnet \
  --resource-group learn-azure-rg \
  --location eastus \
  --address-prefix 10.0.0.0/16 \
  --subnet-name demo-subnet \
  --subnet-prefix 10.0.1.0/24


这会创建一个名为 demo-vnet 的虚拟网络和一个 demo-subnet 子网。

任务 2：创建网络安全组（NSG）
az network nsg create \
  --resource-group learn-azure-rg \
  --name demo-nsg


添加允许 SSH 端口 22 的规则：

az network nsg rule create \
  --resource-group learn-azure-rg \
  --nsg-name demo-nsg \
  --name AllowSSH \
  --protocol tcp \
  --priority 1000 \
  --destination-port-range 22 \
  --access allow

任务 3：创建公有 IP 地址
az network public-ip create \
  --resource-group learn-azure-rg \
  --name demo-ip \
  --sku Basic \
  --allocation-method Dynamic

任务 4：创建网络接口 (NIC)
az network nic create \
  --resource-group learn-azure-rg \
  --name demo-nic \
  --vnet-name demo-vnet \
  --subnet demo-subnet \
  --network-security-group demo-nsg \
  --public-ip-address demo-ip

任务 5：创建虚拟机

方式 1：使用 SSH 密钥（推荐）

az vm create \
  --resource-group learn-azure-rg \
  --name demo-vm \
  --image Ubuntu2204 \
  --admin-username azureuser \
  --authentication-type ssh \
  --ssh-key-values ~/.ssh/id_rsa.pub \
  --nics demo-nic \
  --size Standard_B1s


方式 2：使用密码

az vm create \
  --resource-group learn-azure-rg \
  --name demo-vm \
  --image Ubuntu2204 \
  --admin-username azureuser \
  --admin-password "P@ssword123!" \
  --nics demo-nic

任务 6：连接虚拟机

获取 VM 公网 IP：

az vm show --resource-group learn-azure-rg --name demo-vm -d --query publicIps -o tsv


SSH 登录：

ssh azureuser@<your-public-ip>


验证：

uname -a
lsb_release -a


你应该能看到 Ubuntu 系统信息。

任务 7：验证 NSG 防护

尝试关闭 SSH：

az network nsg rule update \
  --resource-group learn-azure-rg \
  --nsg-name demo-nsg \
  --name AllowSSH \
  --access deny


再次尝试 SSH 登录，应该失败，证明 NSG 规则生效。

任务 8：清理资源
az group delete --name learn-azure-rg --yes --no-wait

🧩 三、验证成果

✅ 你应能完成以下成果：

成功创建 VNet、子网、NSG、公网 IP、NIC、VM；

通过 SSH 登录 VM；

理解 NSG 规则控制；

熟悉资源之间的依赖关系；

掌握命令式 VM 创建流程。

🧠 四、延伸与思考

性能实验
使用不同的 VM Size (Standard_B1s, Standard_D2s_v3) 创建虚拟机，对比价格与性能差异：

az vm list-sizes --location eastus --output table


自定义 VNet 设计
尝试创建两个子网：

frontend-subnet（暴露公网 IP）

backend-subnet（仅内部通信）

入站安全最佳实践

仅允许 SSH 来自特定 IP；

生产环境中建议使用 JumpBox（跳板机）；

使用 Azure Bastion 实现浏览器内安全连接。

官方推荐阅读
Azure 虚拟机快速入门（Microsoft Learn）

📘 今日总结

你学会了如何构建虚拟网络（VNet + Subnet + NSG）；

掌握了使用 Azure CLI 创建 Linux 虚拟机的完整过程；

理解了 SSH 登录机制与安全规则配置；

能够验证网络连通性与安全策略效果；

具备了 IaaS 层的基础实操能力，为后续部署 Web 服务奠定基础。