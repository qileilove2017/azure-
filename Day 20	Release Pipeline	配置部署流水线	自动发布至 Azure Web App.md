🧠 一、今天要搞清楚的三个概念

先用很短的话把今天的关键词说清楚：

CI（Continuous Integration）：代码一提交，就自动构建 + 测试 + 打包（你在 Day 17、18 已经搞定）。

CD（Continuous Delivery / Deployment）：构建完成后，自动部署到目标环境（今天要做的重点）。

Release Pipeline：从“构建产物”到“线上环境”的一条自动化发布链路。

我们会用 YAML 多阶段 Pipeline（推荐做法） 来实现：

Build 阶段：代码构建 + 测试 + 发布 Artifact
Deploy 阶段：把 Artifact 部署到 Azure Web App

🧩 二、前置准备（环境假设）

下面内容默认你有：

一个 Azure 订阅（能创建 Web App）。

一个 Azure DevOps 项目（前面已建立）。

一个已经能产出 Artifact 的 CI Pipeline（类似 Day 18）：

比如 Node.js 构建后有 node-dist.zip

或者 Python 打包后有 python-dist 目录。

接下来我们要加的是：从 Artifact → 部署到 Azure Web App。

⚙️ 三、步骤 1：创建 Azure Web App（如果还没有）

如果你还没建 Web App，可以用 CLI 建一个最简单的（Node 示例）：

# 创建资源组
az group create -n webapp-rg -l eastus

# 创建 App Service Plan（Linux，按需计费）
az appservice plan create \
  -g webapp-rg \
  -n webapp-plan \
  --sku B1 \
  --is-linux

# 创建 Web App（Node 运行时仅作为示例，实际可以是任意语言）
az webapp create \
  -g webapp-rg \
  -p webapp-plan \
  -n my-devops-webapp-123 \
  --runtime "NODE|18-lts"


记住这个名字：
my-devops-webapp-123（后面发布要用）。

🌉 四、步骤 2：在 Azure DevOps 建立 Service Connection

让 Azure DevOps 拿到访问 Azure 的“门票”。

路径：
Project Settings → Service connections → New service connection → Azure Resource Manager

推荐选项：

连接类型：Service principal (automatic)

作用范围：Subscription / Resource Group

勾选 “Grant access permission to all pipelines”

完成后会得到一个连接名，例如：
azure-sp-connection

这个名字等会要在 YAML 里用。

🧱 五、步骤 3：改造为多阶段 Pipeline（CI + CD 一体）

下面给一个完整的 CI + CD 多阶段 YAML 示例，假设你是 Node.js 项目，并且在构建阶段产出 node-dist.zip 作为 Artifact。

你可以把之前的 CI YAML 改成类似这样（比如 .azure-pipelines/ci-cd-webapp.yml）：

trigger:
- main

pool:
  vmImage: ubuntu-latest

stages:
# =====================
# Stage 1: Build & Test
# =====================
- stage: Build
  displayName: 'Build & Test'
  jobs:
  - job: BuildJob
    displayName: 'Build Node.js app'
    steps:
    - task: NodeTool@0
      inputs:
        versionSpec: '18.x'
      displayName: 'Use Node.js 18'

    - script: |
        npm install
      displayName: 'Install dependencies'

    - script: |
        npm test
      displayName: 'Run tests'

    - script: |
        npm run build
      displayName: 'Build app'

    # 假设 build 输出到 dist/，打包成 zip
    - task: ArchiveFiles@2
      inputs:
        rootFolderOrFile: 'dist'
        includeRootFolder: false
        archiveFile: '$(Build.ArtifactStagingDirectory)/node-dist.zip'
        replaceExistingArchive: true
      displayName: 'Archive dist to zip'

    - task: PublishBuildArtifacts@1
      inputs:
        PathtoPublish: '$(Build.ArtifactStagingDirectory)'
        ArtifactName: 'drop'
        publishLocation: 'Container'
      displayName: 'Publish artifact'

# =====================
# Stage 2: Deploy to Web App
# =====================
- stage: Deploy
  displayName: 'Deploy to Azure Web App'
  dependsOn: Build
  condition: succeeded()
  jobs:
  - deployment: DeployWeb
    displayName: 'Deploy to WebApp'
    environment: 'prod'   # 可以在 DevOps 里建 environment 做审批
    strategy:
      runOnce:
        deploy:
          steps:
          - download: current
            artifact: drop

          - task: AzureWebApp@1
            displayName: 'Deploy to Azure Web App'
            inputs:
              azureSubscription: 'azure-sp-connection' # 你的 Service Connection 名
              appType: 'webAppLinux'
              appName: 'my-devops-webapp-123'          # 你的 Web App 名称
              package: '$(Pipeline.Workspace)/drop/node-dist.zip'


这个 Pipeline 做了几件关键事：

Build 阶段：

安装依赖、跑测试、构建、打包为 node-dist.zip

发布为名为 drop 的 Artifact。

Deploy 阶段：

依赖 Build 阶段（dependsOn: Build）

下载刚才叫 drop 的 Artifact

用 AzureWebApp@1 任务把 zip 部署到 my-devops-webapp-123

🧪 六、步骤 4：跑一遍，验证发布是否成功

把上面的 YAML 提交到 repo：

git add .azure-pipelines/ci-cd-webapp.yml
git commit -m "Day20: add CI/CD pipeline to Web App"
git push


在 Azure DevOps → Pipelines 中，选择这个新建的 pipeline，Run 一次。

观察：

Stage Build 是否成功（安装、测试、打包没报错）。

Stage Deploy 是否成功（AzureWebApp 任务绿色）。

部署成功后，在浏览器访问：

https://my-devops-webapp-123.azurewebsites.net


如果你的前端入口文件正确配置，你应该能看到应用页面，或者你在 build 时可以写一行简单的测试输出。

🧰 七、如果是 Python / 其他语言，大致怎么改？

只要你能做到两件事，语言都不重要：

Build 阶段：产出某个目录或 zip 包作为 Artifact；

Deploy 阶段：用合适的 Azure DevOps 任务把包丢到 Web App。

例如 Python（最简单粗暴）：

Build 阶段：把整个项目打成 zip，设为 python-drop.zip

Deploy 阶段：

- task: AzureWebApp@1
  inputs:
    azureSubscription: 'azure-sp-connection'
    appType: 'webAppLinux'
    appName: 'my-python-webapp'
    package: '$(Pipeline.Workspace)/drop/python-drop.zip'


本质完全一样。

🔐 八、结合 Day 19 的 Secret 管理（稍微安全一点）

如果你在部署时需要用到某些配置，比如：

数据库连接字符串

第三方 API key

推荐做法：

在 Key Vault 里存好（Day 19 已学）。

在 DevOps 里通过 Variable Group + Key Vault 引用。

在 Deploy 阶段往 Web App 写入 App Settings。

示例（在 Deploy 阶段加一步）：

- task: AzureAppServiceSettings@1
  displayName: 'Set App Settings'
  inputs:
    azureSubscription: 'azure-sp-connection'
    appName: 'my-devops-webapp-123'
    appSettings: |
      [
        {
          "name": "API_TOKEN",
          "value": "$(api-token)",
          "slotSetting": false
        },
        {
          "name": "DB_PASSWORD",
          "value": "$(db-password)",
          "slotSetting": false
        }
      ]


这样 Web App 的环境变量就从 Key Vault 安全注入了。

✅ 九、今天你应该达成的成果

如果这一节你跟着做完，应该能做到：

有一个多阶段 YAML：Build + Deploy。

每次提交到 main：

自动触发 CI（构建 + 测试 + 打包）。

成功后自动触发 CD（部署到 Azure Web App）。

可以在浏览器直接访问部署结果。

部署过程完全自动化，不需要手点发布。

总结成一句话：
👉 你现在已经有了一条从 Git 提交 → 编译 → 测试 → 打包 → 部署 Web App 的完整 Azure DevOps Release Pipeline。