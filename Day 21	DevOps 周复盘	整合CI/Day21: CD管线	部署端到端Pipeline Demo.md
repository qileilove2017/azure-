Day 21：DevOps 周复盘

主题：DevOps 周复盘｜学习目标：整合 CI/CD 管线｜实战任务：部署端到端 Pipeline Demo

🧠 一、Day 21 的核心目标（先说清楚）

用一句话定义今天：
👉 从一次 git push 开始，到应用成功部署到 Azure Web App，全流程无人介入、自动完成。

你今天要交付的不是“代码”，而是一个完整 DevOps 能力证明：

Git Commit
  ↓
CI（Build + Test）
  ↓
Artifact
  ↓
CD（Deploy）
  ↓
Azure Web App Running


这条链路跑通，你已经具备 初级–中级 DevOps Engineer 的完整闭环能力。

🧩 二、你已经具备的能力（来自 Day 15–20）

在开始之前，我们快速对齐你已经掌握的内容：

天数	能力
Day 15	Azure DevOps 组织、项目、服务结构
Day 16	Azure Repos、Git 分支、PR
Day 17	CI Pipeline（Node / Python）
Day 18	自动化测试 + Artifact
Day 19	Library + Key Vault 管理 Secret
Day 20	多阶段 Pipeline，部署到 Web App

👉 Day 21 的任务：把这些拼成一个“演示级 Demo”。

🧱 三、Demo 架构设计（你要“讲得清楚”的那种）

这是你今天要搭出来、并能解释清楚的架构：

Developer
   │
   │ git push (main)
   ▼
Azure Repos
   │
   ▼
Azure Pipelines (YAML)
   ├── Stage 1: CI
   │     ├── install
   │     ├── test
   │     ├── build
   │     └── publish artifact
   │
   └── Stage 2: CD
         ├── download artifact
         ├── inject secrets (Key Vault)
         └── deploy to Azure Web App


关键词（面试/汇报必说）：

CI / CD

Artifact

Multi-stage pipeline

Secret management

Zero-touch deployment

⚙️ 四、端到端 Pipeline Demo（推荐最终版 YAML）

下面是一份你可以直接作为最终 Demo 使用的 Pipeline
（Node.js 示例，Python 逻辑完全一致）

📁 文件路径
.azure-pipelines/ci-cd-demo.yml

✅ 完整 CI + CD Pipeline（Demo 级）
trigger:
- main

variables:
- group: prod-secrets   # 来自 Day 19：Key Vault 绑定的 Variable Group

pool:
  vmImage: ubuntu-latest

stages:

# ======================
# Stage 1: Continuous Integration
# ======================
- stage: CI
  displayName: 'CI - Build & Test'
  jobs:
  - job: Build
    displayName: 'Build and Test App'
    steps:
    - task: NodeTool@0
      inputs:
        versionSpec: '18.x'

    - script: npm install
      displayName: 'Install dependencies'

    - script: npm test
      displayName: 'Run tests'

    - script: npm run build
      displayName: 'Build application'

    - task: ArchiveFiles@2
      inputs:
        rootFolderOrFile: 'dist'
        includeRootFolder: false
        archiveFile: '$(Build.ArtifactStagingDirectory)/app.zip'
      displayName: 'Package build output'

    - task: PublishBuildArtifacts@1
      inputs:
        PathtoPublish: '$(Build.ArtifactStagingDirectory)'
        ArtifactName: 'drop'
      displayName: 'Publish artifact'

# ======================
# Stage 2: Continuous Deployment
# ======================
- stage: CD
  displayName: 'CD - Deploy to Azure Web App'
  dependsOn: CI
  condition: succeeded()

  jobs:
  - deployment: DeployWeb
    displayName: 'Deploy Application'
    environment: 'prod'
    strategy:
      runOnce:
        deploy:
          steps:
          - download: current
            artifact: drop

          - task: AzureWebApp@1
            displayName: 'Deploy to Web App'
            inputs:
              azureSubscription: 'azure-sp-connection'
              appType: 'webAppLinux'
              appName: 'my-devops-webapp-123'
              package: '$(Pipeline.Workspace)/drop/app.zip'

🔐 五、Secret 注入（把 Day 19 真正用起来）

你应该能明确解释这一点：

Secret 不在代码里

不在 YAML 明文里

来自 Azure Key Vault

通过 Variable Group 注入

例如在应用中使用：

Node.js
console.log(process.env.API_TOKEN);

Python
import os
print(os.environ["DB_PASSWORD"])


Pipeline 中不需要任何额外代码，只要：

variables:
- group: prod-secrets

🧪 六、Day 21 的“验收标准”（非常重要）

如果这是一次真实工作或面试，你今天的 验收标准 是：

✅ 功能层面

git push 到 main

Pipeline 自动触发

CI 阶段成功（test + build）

Artifact 正常生成

CD 阶段自动部署

Web App 页面更新

✅ DevOps 能力层面

你能清楚解释 CI / CD 区别

你知道 Artifact 的作用

你能说清楚 Secret 为什么不能进代码

你能画出 Pipeline 流程图（哪怕用手）

🧠 七、复盘：你这 1 周真正学会了什么？

这是 Day 21 最重要的一部分。

你现在已经掌握：

用 Git + PR 驱动交付

用 YAML 定义流水线（Pipeline as Code）

用 自动化测试 保证质量

用 Artifact 解耦构建与部署

用 Key Vault 管理敏感信息

用 Azure Web App 实现自动部署

这已经是真实企业 DevOps 的最小闭环。