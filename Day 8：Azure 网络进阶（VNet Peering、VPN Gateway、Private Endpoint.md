ay 8：Azure 网络进阶（VNet Peering、VPN Gateway、Private Endpoint）
模块	内容
主题	Azure 网络进阶：VNet Peering、VPN Gateway、Private Endpoint
学习目标	理解多网络互联、混合云连接和私有访问机制；掌握跨 VNet 通信与私有化安全访问配置。
实战任务	创建两个虚拟网络并建立 Peering，配置虚拟网络网关与私有终结点，实现安全通信。
🧠 一、核心知识讲解
1. Azure 虚拟网络通信模型

Azure 网络是完全隔离的，默认情况下：

不同 VNet 之间 不能互通；

不同订阅或区域的 VNet 也互不连接；

必须显式创建 Peering（对等互连） 才能通信。

2. VNet Peering（虚拟网络对等互连）

VNet Peering 是两个虚拟网络之间的“专线桥梁”。
它能让两个网络像一个大内网一样互通，不经公网，不改变 IP 结构。

VNet-A (10.0.0.0/16) ─── Peering ─── VNet-B (10.1.0.0/16)


特点：

延迟低（数据走微软骨干网）

无需网关

适用于同一区域或跨区域（Global Peering）

常见场景：

开发区与生产区互通；

应用层访问数据库层；

多部门网络整合。

3. VPN Gateway（虚拟专用网络网关）

VPN Gateway 是连接 Azure 与本地数据中心 的通道。
它支持：

Site-to-Site VPN（站点对站点）

Point-to-Site VPN（远程个人设备连接）

VNet-to-VNet VPN

VPN 通过 IPsec 隧道 实现加密通信。

    On-premises Router
            │
            ▼
    [Internet]
            │
    Azure VPN Gateway → VNet


💡 在企业场景中，常将 VPN Gateway 部署在核心 VNet 中，用于连接多区域网络或本地 IDC。

4. Private Endpoint（私有终结点）

Private Endpoint 能让 Azure 服务（例如 Storage、SQL、WebApp）
通过 VNet 内部私有 IP 访问，而无需走公网。

VNet Subnet → Private Endpoint → Storage Account (私有访问)


优点：

避免公网暴露；

提高数据安全性；

可配合 NSG、Firewall 实现零信任架构。

🧰 二、实战任务
任务 1：创建两个虚拟网络（VNet-A / VNet-B）
az group create --name network-lab-rg --location eastus

az network vnet create \
  --resource-group network-lab-rg \
  --name VNet-A \
  --address-prefix 10.0.0.0/16 \
  --subnet-name subnet-A \
  --subnet-prefix 10.0.1.0/24

az network vnet create \
  --resource-group network-lab-rg \
  --name VNet-B \
  --address-prefix 10.1.0.0/16 \
  --subnet-name subnet-B \
  --subnet-prefix 10.1.1.0/24

任务 2：创建 VNet Peering

在两个 VNet 之间建立双向连接：

az network vnet peering create \
  --name AtoB \
  --resource-group network-lab-rg \
  --vnet-name VNet-A \
  --remote-vnet VNet-B \
  --allow-vnet-access

az network vnet peering create \
  --name BtoA \
  --resource-group network-lab-rg \
  --vnet-name VNet-B \
  --remote-vnet VNet-A \
  --allow-vnet-access


验证：

az network vnet peering list --resource-group network-lab-rg --vnet-name VNet-A --output table


💡 提示：
在 Portal → 虚拟网络 → 对等互连，可看到状态 “Connected”。

任务 3：验证 VNet 互通性

创建两台 VM，分别放入不同网络：

az vm create \
  --resource-group network-lab-rg \
  --name vmA \
  --vnet-name VNet-A \
  --subnet subnet-A \
  --image Ubuntu2204 \
  --admin-username azureuser \
  --generate-ssh-keys

az vm create \
  --resource-group network-lab-rg \
  --name vmB \
  --vnet-name VNet-B \
  --subnet subnet-B \
  --image Ubuntu2204 \
  --admin-username azureuser \
  --generate-ssh-keys


登录 vmA，ping vmB 的私有 IP（Azure 默认不允许 ICMP，可用 curl 或 netcat 测试）：

nc -zv 10.1.1.4 22


如返回 “succeeded”，说明两台虚拟机网络互通。

任务 4：（进阶）创建 VPN Gateway（了解）

⚠️ 注意：此任务创建时间较长（30–40分钟），可在 Portal 实操。

核心步骤：

创建一个 Gateway Subnet（专用网关子网）：

az network vnet subnet create \
  --resource-group network-lab-rg \
  --vnet-name VNet-A \
  --name GatewaySubnet \
  --address-prefix 10.0.255.0/27


创建公用 IP：

az network public-ip create \
  --resource-group network-lab-rg \
  --name vpn-gateway-ip \
  --allocation-method Dynamic


创建 VPN Gateway：

az network vnet-gateway create \
  --name vpn-gateway \
  --public-ip-address vpn-gateway-ip \
  --resource-group network-lab-rg \
  --vnet VNet-A \
  --gateway-type Vpn \
  --sku VpnGw1 \
  --vpn-type RouteBased

任务 5：创建 Private Endpoint（访问 Storage）

创建一个存储账户：

az storage account create \
  --name mystoragepe$RANDOM \
  --resource-group network-lab-rg \
  --location eastus \
  --sku Standard_LRS \
  --kind StorageV2


创建 Private Endpoint：

az network private-endpoint create \
  --name pe-storage \
  --resource-group network-lab-rg \
  --vnet-name VNet-A \
  --subnet subnet-A \
  --private-connection-resource-id $(az storage account show --name mystoragepe123 --query id -o tsv) \
  --group-id blob


验证：

在 Portal → Storage Account → 网络 → “私有终结点连接”；

查看已分配的私有 IP。

💡 现在，你可以在 VNet 内通过私有 IP 访问 Blob Storage，而无需公网访问。

任务 6：（可选）关闭 Storage 公网访问
az storage account update \
  --name mystoragepe123 \
  --public-network-access Disabled

🧩 三、验证成果

✅ 你应能完成以下验证：

成功建立 VNet Peering（A ↔ B 双向）；

两台 VM 在不同网络中可互通；

VPN Gateway 了解创建过程；

创建 Storage Private Endpoint；

关闭 Storage 公网后仍可从内部访问。

🧠 四、延伸与思考

对等互连设计建议

保持地址空间不重叠；

避免环状 Peering（可能导致路由混乱）；

可通过 Hub-Spoke 模型集中管理网络。

Private Endpoint 安全最佳实践

禁止公网访问；

搭配 Azure Firewall 监控流量；

在生产环境中使用专用子网隔离私有端点。

Hub-Spoke 网络拓扑图

            VPN Gateway / Firewall
                    │
                Hub VNet
            ┌───────────────┐
            │    Peering    │
            └───────────────┘
            /               \
        Spoke1 VNet        Spoke2 VNet
        (App Layer)          (DB Layer)


这是企业网络的标准设计 —— 中心 Hub 负责流量与安全策略，Spoke 网络承载不同业务。

📘 今日总结

你掌握了 Azure 网络的高级能力：VNet Peering、VPN Gateway、Private Endpoint；

理解了如何实现多网络互通与混合云连接；

学会了如何让 Azure 服务通过私有 IP 访问，提升安全性；

熟悉了企业常用的 Hub-Spoke 网络拓扑；

具备了设计安全、互联、可扩展网络架构的基础能力。