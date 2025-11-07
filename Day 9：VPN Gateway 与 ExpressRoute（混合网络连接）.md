🌐 Day 9：VPN Gateway 与 ExpressRoute（混合网络连接）
模块	内容
主题	VPN Gateway、ExpressRoute
学习目标	理解 Azure 与本地数据中心的混合网络连接方式；掌握 VPN Gateway 与 ExpressRoute 的架构与配置方法。
实战任务	设计并配置虚拟专用网关，模拟混合网络连接，了解私有线路 ExpressRoute 的企业应用场景。
🧠 一、核心知识讲解
1. 什么是混合云网络连接

在企业架构中，Azure 通常不会独立存在，而是与本地数据中心（On-Premises）互联，形成“混合云”：

On-Premises ↔ VPN / ExpressRoute ↔ Azure VNet


两种主要连接方式：

VPN Gateway：通过公网（IPSec隧道）实现加密连接；

ExpressRoute：通过运营商专线实现物理私有连接。

2. VPN Gateway 架构与原理

Azure VPN Gateway 是一种部署在 VNet 中的网络资源，用于与其他网络建立安全隧道。

类型：

Site-to-Site (S2S)：连接企业本地路由器与 Azure。

Point-to-Site (P2S)：单个客户端远程接入。

VNet-to-VNet：连接不同区域的两个 VNet。

基本架构：

        On-Premises Router
            │
        (IPSec Tunnel)
            │
        VPN Gateway
            │
        Azure Virtual Network


💡 特点：

使用公网通信；

支持加密 (IKEv2/IPSec)；

成本较低，部署灵活；

适合开发 / 测试 / 中小企业环境。

3. ExpressRoute 概念与优势

ExpressRoute 是 Azure 提供的 专线级私有连接服务，可通过合作运营商建立本地机房与 Azure 的高速链路。

        项目	VPN Gateway	ExpressRoute
        连接介质	公网隧道 (IPSec)	专线（物理私网）
        带宽	10 Mbps ~ 1 Gbps	50 Mbps ~ 100 Gbps
        延迟	相对较高（公网）	极低（专用光纤）
        SLA	无保证	99.95% 可用性
        场景	中小型企业 / 测试	金融、制造业、核心系统

💡 ExpressRoute 常用于：

金融合规性要求（数据不走公网）；

多区域数据复制；

大规模低延迟通信。

⚙️ 二、实战任务

⚠️ 以下 VPN Gateway 实战可使用 Azure 自带模拟网络；ExpressRoute 为企业服务，通常需运营商配合（理论学习为主）。

任务 1：创建虚拟网络与子网
az group create --name hybrid-lab-rg --location eastus

az network vnet create \
  --resource-group hybrid-lab-rg \
  --name hybrid-vnet \
  --address-prefix 10.0.0.0/16 \
  --subnet-name default \
  --subnet-prefix 10.0.1.0/24

任务 2：创建专用网关子网

VPN Gateway 必须放在一个名为 GatewaySubnet 的子网内。

az network vnet subnet create \
  --resource-group hybrid-lab-rg \
  --vnet-name hybrid-vnet \
  --name GatewaySubnet \
  --address-prefix 10.0.255.0/27

任务 3：创建公网 IP 与 VPN Gateway
az network public-ip create \
  --resource-group hybrid-lab-rg \
  --name vpn-gateway-ip \
  --allocation-method Dynamic

az network vnet-gateway create \
  --name vpn-gateway \
  --resource-group hybrid-lab-rg \
  --vnet hybrid-vnet \
  --public-ip-address vpn-gateway-ip \
  --gateway-type Vpn \
  --vpn-type RouteBased \
  --sku VpnGw1


⚙️ 该过程需约 30 分钟。

任务 4：创建本地网关（模拟本地网络）
az network local-gateway create \
  --resource-group hybrid-lab-rg \
  --name onpremise-gw \
  --gateway-ip-address 13.82.101.24 \
  --local-address-prefixes 192.168.0.0/16

任务 5：建立站点到站点连接 (S2S)
az network vpn-connection create \
  --name s2s-connection \
  --resource-group hybrid-lab-rg \
  --vnet-gateway1 vpn-gateway \
  --local-gateway2 onpremise-gw \
  --shared-key "Hybrid2025"


💡 若有第二个 Azure 区域，可用同样命令配置 VNet-to-VNet VPN。

任务 6：（理论）ExpressRoute 连接流程

联系 Azure 官方或运营商（Equinix、China Telecom、Megaport）；

申请 ExpressRoute Circuit（专线电路）；

运营商建立物理连接；

在 Azure 中配置 ExpressRoute 资源：

az network express-route create \
  --name er-circuit \
  --resource-group hybrid-lab-rg \
  --bandwidth 200 \
  --provider "Equinix" \
  --peering-location "New York" \
  --sku-family MeteredData \
  --sku-tier Standard


建立路由表（BGP Peering）；

验证连接状态：

az network express-route list --output table

🧩 三、验证成果

✅ 你应能完成：

成功创建 VPN Gateway 并模拟混合连接；

理解本地与 Azure 之间的站点到站点架构；

掌握 ExpressRoute 创建与用途；

能绘制完整的混合云网络拓扑图。

🧠 四、延伸与思考

安全设计建议

使用 IKEv2 加密算法；

启用网络流量监控；

搭配 Azure Firewall 或 NSG；

使用 Azure Bastion 避免公网 SSH。

混合云架构常见拓扑

        On-Premises Network
            │
        [VPN / ExpressRoute]
            │
        Hub VNet (EastUS)
            ├── Firewall / GatewaySubnet
            ├── Spoke VNet 1 (App)
            └── Spoke VNet 2 (Database)


💡 Hub-Spoke 模型 是企业标准架构：

Hub 集中流量、安全与路由；

Spoke 用于不同业务系统的隔离与互通。

ExpressRoute 高可用性设计

建立双线路（主备电路）；

各接入点跨区域；

启用 FastPath 提高吞吐量。

成本与性能比较

项目	VPN Gateway	ExpressRoute
成本	低（月均约 $100）	高（月均 $1,000+）
延迟	较高（公网依赖）	低（专用光纤）
带宽	最高 1 Gbps	最高 100 Gbps
适用场景	中小企业、PoC 测试	金融、电信、跨区域复制
📘 今日总结

理解了 VPN Gateway 与 ExpressRoute 的概念与区别；

掌握了 S2S、VNet-to-VNet 连接配置；

了解了 ExpressRoute 的高可用设计；

能够绘制并解释混合云架构；

为企业级混合网络设计与安全治理打下基础。