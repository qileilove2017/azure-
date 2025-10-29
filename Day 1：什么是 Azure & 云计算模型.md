

## 🌤️ Day 1：什么是 Azure & 云计算模型（IaaS / PaaS / SaaS）

| 模块       | 内容                                                                 |
| -------- | ------------------------------------------------------------------ |
| **主题**   | 什么是 Azure & 云计算模型（IaaS / PaaS / SaaS）                              |
| **学习目标** | 理解云计算的基本概念，熟悉 Azure 的整体架构与核心服务分类。掌握 IaaS、PaaS、SaaS 三种服务模型的区别与应用场景。 |
| **实战任务** | 注册 Azure 免费账号，登录 Azure Portal，熟悉控制台结构与主要服务入口。                      |

---

### 🧠 一、核心知识讲解

#### 1. 云计算的三种服务模型

| 模型       | 英文全称                        | 代表服务        | 使用者责任           | 示例                                       |
| -------- | --------------------------- | ----------- | --------------- | ---------------------------------------- |
| **IaaS** | Infrastructure as a Service | 提供虚拟机、存储、网络 | 用户管理操作系统、应用、中间件 | Azure Virtual Machines、VNet、Blob Storage |
| **PaaS** | Platform as a Service       | 提供开发平台、运行环境 | 用户专注应用代码        | Azure App Service、Azure SQL Database     |
| **SaaS** | Software as a Service       | 提供完整的应用软件   | 用户只需使用          | Microsoft 365、Dynamics 365、Power BI      |

💡 **理解方式：**

* IaaS = “云上的服务器”
* PaaS = “云上的开发平台”
* SaaS = “云上的应用软件”

---

#### 2. Azure 的核心服务结构

Azure 提供超过 200 种服务，核心分为六大类：

* **计算 (Compute)**：Virtual Machines、App Services、Functions、AKS
* **存储 (Storage)**：Blob、Files、Queue、Table
* **网络 (Networking)**：VNet、Load Balancer、CDN、ExpressRoute
* **数据库 (Database)**：Azure SQL、Cosmos DB、PostgreSQL、MySQL
* **AI & 分析 (AI + Analytics)**：Azure OpenAI、Cognitive Services、Synapse
* **安全与治理 (Security & Management)**：Azure AD、Policy、Monitor、Cost Management

---

#### 3. Azure 全局架构

Azure 的资源分层如下：

```
订阅 (Subscription)
└── 资源组 (Resource Group)
    ├── 虚拟机 (VM)
    ├── 存储账户 (Storage Account)
    ├── 数据库 (SQL Database)
    └── 网络 (VNet)
```

一个资源组包含多个服务实例，统一计费与访问控制。

---

### 🧰 二、实战任务

#### 任务 1：注册 Azure 免费账号

1. 打开 [Azure 官网](https://azure.microsoft.com/zh-cn/free/)
2. 使用 Microsoft 账号登录。
3. 注册免费试用（通常赠送 $200 美元信用额度，有效期 30 天）。
4. 进入 **Azure Portal**： [https://portal.azure.com](https://portal.azure.com)

---

#### 任务 2：熟悉 Azure Portal 界面

登录后，你会看到：

* **左侧导航栏**：服务入口（虚拟机、存储、SQL、Monitor等）
* **搜索栏**：可快速搜索任何资源
* **主页卡片**：最近使用的服务、订阅状态、快捷创建入口

建议：

* 将常用服务（如 Virtual Machines、Storage Accounts）固定到主页。
* 尝试切换“深色模式”和“个性化布局”。

---

#### 任务 3：使用 Azure CLI 创建资源组

Azure CLI 是操作 Azure 的命令行工具，适合开发者使用。

**安装命令（macOS 示例）**

```bash
brew update && brew install azure-cli
```

**登录**

```bash
az login
```

这会自动打开浏览器并要求输入 Microsoft 账号授权。

**创建资源组**

```bash
az group create --name learn-azure-rg --location eastus
```

**验证资源组是否创建成功**

```bash
az group list --output table
```

---

### 🧩 三、验证成果

完成后，你应能：

* 登录 Azure Portal；
* 看到 “learn-azure-rg” 资源组；
* 确认 CLI 登录成功；
* 理解 IaaS / PaaS / SaaS 的区别。

✅ **成果截图建议：**

* Portal 首页（带资源组）；
* CLI 输出结果。

---

### 🪜 四、进阶延伸（可选）

1. 学习 Azure CLI 的更多命令：

   ```bash
   az resource list
   az account show
   az configure --defaults location=eastus
   ```
2. 了解 Azure 全球区域（Regions）与可用区（Availability Zones）的区别。
3. 阅读官方入门教程：
   [Azure Fundamentals 学习路径（Microsoft Learn）](https://learn.microsoft.com/zh-cn/training/paths/azure-fundamentals/)

---

### 📘 今日总结

* 你已了解云计算的三大模型 IaaS / PaaS / SaaS；
* 理解了 Azure 的整体架构与核心服务；
* 完成了免费账号注册与第一个资源组创建；
* 熟悉了 Portal 和 CLI 的基本操作。
