Day 18：测试与 Artifact 管理

主题：测试与 Artifact 管理｜学习目标：引入自动化测试｜实战任务：构建 + 测试 + 发布包

🧠 一、核心知识（必学）
1. 什么是 Artifact？

Artifact 是 CI 过程中生成的“构建产物”，例如：

Python：打包成 .whl、.tar.gz

Node.js：dist/ 文件夹、.zip

Java：.jar、.war

前端项目：build/ 文件夹

文档生成：HTML 文档.zip

Artifact 必须可追溯、可下载、可用于部署。

在 Azure DevOps 中：

Build pipeline 产出 → Artifact
Release pipeline 或 CD pipeline 使用 → Artifact

2. CI 三大步骤（今天全部实现）
1. 构建（Build）
2. 测试（Test）
3. 发布 Artifacts


企业级 CI 中：
若测试失败 → 不发布 artifact
若构建失败 → 不发布 artifact
确保产出物 ALWAYS 可部署、可用。

⚙️ 二、今日实践任务

你将完成两个项目：

① Python 项目

运行 pytest

生成 test report

打包为 wheel

上传 artifact

② Node.js 项目

运行 jest

执行 build

压缩 dist/

上传 artifact

🐍 Part 1：Python（构建 + 测试 + 发布 Artifact）
Repo 结构建议：
python-app/
 ├── app.py
 ├── requirements.txt
 ├── setup.py
 └── tests/
      └── test_app.py

setup.py（示例）

如果你没有 setup.py，这是一个通用模板：

from setuptools import setup, find_packages

setup(
    name="python-app-demo",
    version="0.1.0",
    packages=find_packages(),
)

Python CI + Artifact YAML

创建：
.azure-pipelines/python-ci-artifact.yml

trigger:
- main

pool:
  vmImage: ubuntu-latest

steps:
# Step 1: 使用 Python
- task: UsePythonVersion@0
  inputs:
    versionSpec: '3.10'

# Step 2: 安装依赖
- script: |
    python -m pip install --upgrade pip
    pip install -r requirements.txt
    pip install setuptools wheel pytest
  displayName: 'Install dependencies'

# Step 3: 运行测试
- script: |
    pytest --junitxml=test-results.xml
  displayName: 'Run tests'

# Step 4: 发布测试结果
- task: PublishTestResults@2
  inputs:
    testResultsFiles: 'test-results.xml'
    testRunTitle: 'Python Test Results'

# Step 5: 构建 wheel 包
- script: |
    python setup.py sdist bdist_wheel
  displayName: 'Build package'

# Step 6: 发布构建产物
- task: PublishBuildArtifacts@1
  inputs:
    PathtoPublish: 'dist'
    ArtifactName: 'python-dist'
  displayName: 'Publish artifacts'


运行后会看到：

Artifacts:
 └── python-dist/
      ├── python_app_demo-0.1.0-py3-none-any.whl
      └── python_app_demo-0.1.0.tar.gz


这就是标准 Python 部署产物。

🟩 Part 2：Node.js（构建 + 测试 + 打包 + Artifact）
推荐 Repo 目录：
node-app/
 ├── package.json
 ├── index.js
 ├── src/
 ├── dist/
 └── __tests__/
      └── app.test.js

package.json 示例
{
  "name": "node-ci-demo",
  "scripts": {
    "test": "jest",
    "build": "mkdir -p dist && echo 'build output' > dist/output.txt"
  },
  "devDependencies": {
    "jest": "^29.0.0"
  }
}


确保至少有一个 npm run build 的输出文件。

Node.js CI + Artifact YAML

创建：
.azure-pipelines/node-ci-artifact.yml

trigger:
- main

pool:
  vmImage: ubuntu-latest

steps:
# Step 1: Node.js 安装
- task: NodeTool@0
  inputs:
    versionSpec: '18.x'
  displayName: 'Use Node.js 18'

# Step 2: 安装依赖
- script: npm install
  displayName: 'Install dependencies'

# Step 3: 运行测试
- script: npm test -- --ci --reporters=jest-junit
  displayName: 'Run tests'

# Step 4: 发布测试报告
- task: PublishTestResults@2
  inputs:
    testResultsFiles: '**/junit.xml'
    testRunTitle: 'Node.js Test Results'

# Step 5: Build 项目
- script: npm run build
  displayName: 'Build project'

# Step 6: 打包 dist 文件夹
- task: ArchiveFiles@2
  inputs:
    rootFolderOrFile: 'dist'
    includeRootFolder: false
    archiveFile: '$(Build.ArtifactStagingDirectory)/node-dist.zip'

# Step 7: 发布构建产物
- task: PublishBuildArtifacts@1
  inputs:
    PathtoPublish: '$(Build.ArtifactStagingDirectory)'
    ArtifactName: 'node-dist'


构建后你会看到：

node-dist/
   └── node-dist.zip


这个 zip 就可以用于下一步 CD 部署。

🎯 三、你今天应完成的能力（验证）

Day 18 成果 checklist：

✔ 掌握 Azure Pipelines 测试集成
✔ 学会生成测试报告（Python / Node）
✔ 会将构建产物打包为 Artifact
✔ 建立完整的 CI（构建 + 测试 + 发布）
✔ 有两个完整示例（Python 与 Node）
✔ Artifact 可用于 Day 19 的部署（CD）

你正式拥有可复用的 CI Pipeline 模板。

🧠 四、最佳实践（企业级）

不允许测试失败的代码进入 main

不允许无 Artifact 的构建进入 CD stage

每个 Pipeline 都必须有：
🔹 install → 🔹 test → 🔹 build → 🔹 publish

所有 Artifact 必须可溯源（Build ID、commit ID）

推荐将 CI 与 PR 策略绑定（构建必须通过才能合并）

📘 今日总结

你完成了：

自动化测试

打包构建产物

产物归档与发布

可视化测试报告

Python & Node.js 双生态 CI

企业级构建流程完整链路

你现在具备：
“生产级 CI” 能力。