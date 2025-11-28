Day 17：Azure Pipelines（YAML）

主题：Azure Pipelines（YAML）｜学习目标：学习 CI 基础｜实战任务：构建 Node.js/Python 应用的 CI Pipeline

🧠 一、什么是 CI（持续集成）？

CI（Continuous Integration）是现代软件工程的核心思想：

每一次提交代码，都自动构建、自动测试、自动检查质量。

CI 的关键目标：

让代码尽早发现问题

自动运行测试

自动构建项目

自动产出构建结果（Artifacts）

CI 是整个 DevOps 成功的关键步骤。

Azure Pipelines 支持 YAML 模式、跨平台、跨语言、跨云，非常灵活。

🔧 二、Azure Pipelines YAML 的结构（必须理解）

一个 YAML Pipeline 通常包含 4 个部分：

trigger:
pool:
variables:
steps:


完整示例：

trigger:
- main

pool:
  vmImage: ubuntu-latest

steps:
- script: echo Hello

🛠️ 三、实战任务（今天的目标）
你今天将完成以下三个 Pipeline：

Python 项目 CI

Node.js 项目 CI

（可选）CI 产物上传 Artifacts

你会学到：

如何选择构建环境

安装依赖（pip / npm）

运行测试

打包构建产物

🟦 Part 1：创建 Pipeline（入口）

进入：
Azure DevOps → Pipelines → New Pipeline

选择：

Azure Repos Git

选择你的项目仓库

选择 “Starter pipeline”

清空默认 YAML，准备粘贴我们的内容。

🐍 Part 2：Python CI Pipeline（完整示例）

你可以在你的 repo 中先创建一个 Python 项目，比如：

project/
 ├── requirements.txt
 ├── app.py
 └── tests/
      └── test_app.py

requirements.txt 示例
pytest
flask

CI YAML（复制即可用）

创建文件 .azure-pipelines/python-ci.yml：

trigger:
- main

pool:
  vmImage: ubuntu-latest

steps:
- task: UsePythonVersion@0
  inputs:
    versionSpec: '3.10'

- script: |
    python -m pip install --upgrade pip
    pip install -r requirements.txt
  displayName: 'Install dependencies'

- script: |
    pytest -q
  displayName: 'Run Tests'

- task: PublishTestResults@2
  inputs:
    testResultsFiles: '**/test-results.xml'
    testRunTitle: 'Python Test Results'


推送后，会自动触发 CI：

✔ 自动安装依赖
✔ 自动运行 pytest
✔ 自动上传报告

🟩 Part 3：Node.js CI Pipeline（完整示例）

创建一个 Node.js 项目：

project/
 ├── package.json
 ├── index.js
 └── __tests__/
      └── app.test.js

package.json 示例
{
  "name": "ci-demo",
  "scripts": {
    "test": "jest",
    "build": "echo building..."
  },
  "devDependencies": {
    "jest": "^29.0.0"
  }
}

CI YAML（复制即可用）

创建 .azure-pipelines/node-ci.yml：

trigger:
- main

pool:
  vmImage: ubuntu-latest

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
  displayName: 'Run unit tests'

- script: |
    npm run build
  displayName: 'Build application'

- task: PublishBuildArtifacts@1
  inputs:
    PathtoPublish: 'dist'
    ArtifactName: 'node-dist'


构建通过后，会生成：

drop/node-dist/


你可以用于下一步 CD 部署。

🧩 Part 4：（可选）CI 产物上传 Artifacts

如果你想让 pipeline 输出最终构建产物（比如 ZIP 包），可以用：

- task: ArchiveFiles@2
  inputs:
    rootFolderOrFile: '.'
    includeRootFolder: false
    archiveFile: '$(Build.ArtifactStagingDirectory)/build.zip'
    replaceExistingArchive: true

- task: PublishBuildArtifacts@1
  inputs:
    PathtoPublish: '$(Build.ArtifactStagingDirectory)'
    ArtifactName: 'build-zip'

🎯 你今天必须完成的成果（验证）

完成 Day 17 后，你应该能够：

✔ 创建一个 YAML Pipeline
✔ Push 代码后自动触发构建
✔ 构建 Python / Node.js 应用
✔ 自动运行测试并显示结果
✔ Pipeline 100% 成功执行
✔ 看到 Artifacts（构建产物）

你现在已经掌握：

Git（Day16）

CI Pipeline（Day17）

下一步将进入：
🚀 持续交付（CD）

🧠 五、CI 常见失败与解决办法
问题	原因	解决方案
Cannot find module	node_modules 没装	检查 npm install 步骤
pytest 找不到	缺 pip install	检查 requirements.txt
权限不足	build agent 默认权限限制	添加 sudo 或运行容器
某个模块版本冲突	锁文件有问题	删除 node_modules 重新安装
Pipeline 不触发	trigger 配置错	检查 YAML 中的 trigger
📘 今日总结

你已经完成了：

CI 基础

YAML Pipeline 编写

Python CI 流程

Node.js CI 流程

测试集成（pytest / jest）

构建产物生成

自动化构建触发

这一步让你正式具备真实 DevOps 工程师的核心技能。