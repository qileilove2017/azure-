Day 15：Azure DevOps 概览

主题：Azure DevOps 概览｜学习目标：了解 DevOps 服务结构｜实战任务：创建 DevOps 组织与项目

🧠 一、Azure DevOps 是什么？

Azure DevOps = 一套完整的软件工程工作平台
覆盖软件研发的端到端生命周期，从计划到部署到监控。

它包含以下核心五大服务：

服务	说明	用途
Azure Boards	Work Item、需求、迭代管理	项目管理
Azure Repos	Git 仓库、分支策略、PR	代码托管
Azure Pipelines	CI/CD 管道（支持 YAML）	构建 + 部署
Azure Test Plans	测试用例管理	QA 测试
Artifacts	包管理（NuGet、npm、PyPI）	包分发

企业常见使用方式：

需求（Boards）
   ↓
代码（Repos）
   ↓
构建测试（Pipelines）
   ↓
部署（Pipelines）
   ↓
包分发（Artifacts）


Azure DevOps 最大优势：

企业级权限管理（Azure AD）

强大的 CI/CD（支持多云、多语言）

完全可扩展（Webhooks、REST API、Pipeline 扩展）

支持 GitHub、Bitbucket、GitLab 等外部 repo

🔍 二、Azure DevOps 架构层级

整体结构非常清晰：

Organization （组织）
   └── Project （项目）
        ├── Boards
        ├── Repos
        ├── Pipelines
        ├── Test Plans
        └── Artifacts


组织（Org）：公司级别，例如：

yourcompany-devops


项目（Project）：某个产品或团队，例如：

backend-api-project
mobile-app-project
platform-devops-tools

⚙️ 三、实战任务

今天的目标任务是：

任务 1：创建 Azure DevOps Organization（组织）
任务 2：创建 Project（项目）
任务 3：快速浏览 DevOps 各个模块

这三个操作完成后，你就正式拥有一个 Azure DevOps 工作空间。

🛠️ 四、实战任务详细步骤
任务 1：创建 Azure DevOps Organization
1. 打开 Azure DevOps 注册页面

访问：
👉 https://dev.azure.com

使用你的 Microsoft / Azure AD 账号登录。

2. 创建 Organization

点击 Create new organization:

填写：

Organization name：my-azure-devops-org

Geo location：选择 Singapore / Southeast Asia / 根据需要

点击 Continue。

💡 创建完成后地址为：

https://dev.azure.com/my-azure-devops-org/

任务 2：创建 Project（项目）

点击 "New Project"

填写内容：

字段	示例
Project Name	webapp-demo
Description	Demo Web App for learning
Visibility	Private（默认）
Version Control	Git
Work Item process	Basic（推荐）

然后点击 Create。

⏱️ 完成后你应该看到：

左侧导航栏：

Boards（工作项）

Repos（代码）

Pipelines（CI/CD）

Test Plans

Artifacts

并出现默认 README.md。

任务 3：快速浏览 Azure DevOps 各模块

你无需今天做深度配置，只需理解结构。

① Azure Boards

进入：
Boards → Work items

你会看到：

User Story

Task

Bug

点击 “New Work Item” → 创建一个 User Story：

Title: Create basic webapp
Description: First backlog item

② Azure Repos

进入：Repos → Files

你会看到一个默认的空 Git repo。

点击 “Initialize” → 创建 README.md
并生成 main 分支。

💡 你可以在 PC 上 clone：

git clone https://dev.azure.com/my-azure-devops-org/webapp-demo/_git/webapp-demo

③ Azure Pipelines

进入：Pipelines → Pipelines → Create Pipeline

虽然今天不需要创建完整 pipeline，
但你可以点击 “Starter pipeline” 预览 YAML，例如：

trigger:
- main

pool:
  vmImage: ubuntu-latest

steps:
- script: echo "Hello DevOps"
  displayName: 'Run a one-line script'

④ Azure Artifacts

进入：Artifacts

你会看到：

npm

nuget

maven feeds

但我们今天主要了解结构，不做配置。

⑤ Azure Test Plans

浏览测试管理页面。
（Note：企业版需要购买 Test Plan 许可证）

🧪 五、环境验证（你今天应该能做到）

✔ 完成 Azure DevOps 组织创建
✔ 完成 Project 创建
✔ 能看到 Boards、Repos、Pipelines、Artifacts、Test Plans
✔ Repo 已被初始化（有 README）
✔ 能创建一个 Work Item

🧠 六、补充知识：Azure DevOps 与 GitHub Actions 对比
项目	Azure DevOps	GitHub Actions
企业级权限	强	中等
CI/CD	⭐⭐⭐⭐⭐	⭐⭐⭐⭐
项目管理（Boards）	⭐⭐⭐⭐⭐	⭐⭐（依靠 Projects Beta）
包管理	内置	GitHub Packages
测试管理	完整	不支持
企业内部落地成熟度	高	中等

企业若涉及：

安全审计

流程审批

大量 Work Item
则 Azure DevOps 更适合。

📘 今日总结（你掌握了什么？）

今天你已经：

✔ 理解了 Azure DevOps 的五大模块
✔ 创建了完整的 DevOps Organization
✔ 创建并初始化一个 Project
✔ 了解了端到端 DevOps 生命周期
✔ 打通了未来 CI/CD 与 DevOps Pipeline 的基础