Day 16：Azure Repos（Git）

主题：Azure Repos (Git)｜学习目标：掌握 Git 分支策略｜实战任务：推送代码并触发 build

🧠 一、核心知识：企业 Git 分支策略（必会）

Azure Repos 100% 支持 Git，因此今天重点学习 Git Flow / Trunk-based 等策略。

🔧 常见企业级分支策略
① Main（或 master）

产品正式版本所在

受保护，不允许直接 push

每次 release 基于此分支创建 tag

② Develop（可选）

开发主干

所有 feature 分支最终合并到 develop

③ Feature 分支

命名示例：

feature/login-api
feature/user-auth
feature/ui-header


用于单一功能开发。

④ Release 分支

用于准备正式版本：

release/1.2.0

⑤ Hotfix

生产环境修复：

hotfix/urgent-login-bug

🔥 推荐你使用的分支策略（简单实用）

你是一个人学习，因此推荐 简单但企业级可用的策略：

main → feature → PR → main


企业团队通常会开启：

分支保护（PR 必须）

代码审查（至少 1 reviewer）

必须通过 build pipeline 才能合并

⚙️ 二、实战任务

今天你要完成以下四件事：

Clone Azure Repo 到本地

创建 feature 分支

编写简单代码推送回 Repos

自动触发 Pipeline 构建

完成后，你会看到完整的 DevOps 流程成功跑通 💥

🛠️ Part 1：Clone Repo（克隆仓库）

进入 Azure DevOps → Repos → Files
点击 Clone 复制 Git URL：

https://dev.azure.com/<org>/<project>/_git/<repo>


在本地执行：

git clone https://dev.azure.com/<org>/<project>/_git/<repo>
cd <repo>

🛠️ Part 2：创建 Feature 分支

创建分支：

git checkout -b feature/day16-demo


查看当前分支：

git branch

🛠️ Part 3：添加简单代码并推送

创建一个简单的 Python 文件 /hello.py：

print("Hello Azure DevOps Pipeline!")


然后：

git add .
git commit -m "Day16: add hello pipeline script"
git push --set-upstream origin feature/day16-demo


推送成功后，你会在 Azure Repos 看到新的 feature 分支。

🛠️ Part 4：创建 Build Pipeline（自动触发 build）

进入：

Azure DevOps → Pipelines → Create Pipeline

选择：

Azure Repos Git

选择你的 repo

选择 Starter pipeline

将 YAML 替换为以下内容：

trigger:
- main

pr:
- main

pool:
  vmImage: ubuntu-latest

steps:
- script: echo "Running Python Script"
  displayName: "Echo Step"

- script: python3 hello.py
  displayName: "Run Python Script"


点击 Run。

🧪 Part 5：创建 Pull Request（PR）并触发 Build

回到 Azure DevOps → Repos → Pull Requests → New PR

Source：feature/day16-demo

Target：main

提交 Pull Request。

你会看到：

⭐ 自动触发 CI build
⭐ 若成功 → 才能允许合并（如果启用 Branch Policy）

🔐（可选）启用 Branch Policy（强力推荐）

进入：

Repos → Branches → main → Branch Policies

启用以下选项：

Require a minimum number of reviewers = 1

Build validation = Pipeline（必须通过）

Require linked work item = On

Limit merge types = Squash

这样 main 分支就变得非常安全。

🧩 三、你今天应该能做到的结果（验证）

完成 Day 16 后，你应该能成功做到：

✔ 克隆 Azure Repo 到本地
✔ 创建一个 feature 分支
✔ 推送代码到 Azure Repos
✔ 创建 PR
✔ 自动触发 CI Pipeline
✔ 构建成功并完成合并

你现在已经走完真正意义上的 DevOps 基础操作循环。

🧠 四、延伸：企业常用分支策略（理解更深入）
Git Flow（大团队常用）
main
develop
feature/*
release/*
hotfix/*

Trunk-based（DevOps 推荐）
main + 短周期 feature 分支

Feature Workflow（轻量适合你）
main ← feature/*

📘 今日总结

今天你掌握了：

Git 分支策略（Feature → PR → Main）

Azure Repos 使用方式

CI Pipelines 触发机制

通过代码推送触发自动化构建

企业级 Branch Policy 的作用

你现在已经具备 Azure DevOps 实战操作能力，正式进入 DevOps 的核心：持续集成（CI）。