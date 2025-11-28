Day 19：环境变量与 Secret 管理

主题：环境变量与 Secret 管理｜学习目标：管理配置安全｜实战任务：使用 Library + KeyVault 管理 Pipeline 密钥

🧠 一、为什么要管理环境变量与 Secret？

在实际 DevOps 项目中，你通常需要在 CI/CD 时使用：

数据库连接字符串

API Token

Storage Key

Service Principal 密钥

环境特定配置（Prod / Dev）

JWT Secret

第三方 API Key

不能写在代码里！不能写在 YAML 里！不能 commit 到 Git！

所以必须使用 DevOps 的安全机制：

Azure Key Vault → 存密钥  
Azure DevOps Library → 在 CI/CD 中安全注入  
Pipeline 使用 → $(secretName)

🟦 二、Azure DevOps 提供的安全能力
功能	用途
Pipeline Variables	简单变量，不安全（不建议放敏感信息）
Variable Group (Library)	跨 Pipeline 重用，可加密
Secret Variables	Library 中加密存储
Azure Key Vault Integration	最安全的方式（企业强制）
Service Connection	访问 Azure / GitHub / 其他云
🔐 三、企业推荐方式（最安全）
Azure Key Vault（存密钥）
     ↓（Reference）
DevOps Library（Variable Group）
     ↓（通过 pipeline 引用）
Pipeline（使用）

⚙️ 四、今日实战任务（完整流程）

你今天将完成：

① 创建 Azure Key Vault
② 添加 Secrets（API Key / Token 示例）
③ 在 DevOps 中创建 Variable Group
④ 连接 Variable Group ↔ Key Vault
⑤ 在 YAML Pipeline 中使用 Secrets

完成后你将拥有企业级安全变量注入能力。

🛠️ Part 1：创建 Azure Key Vault
az group create -n devops-sec-rg -l eastus

az keyvault create \
  -n devops-kv-$RANDOM \
  -g devops-sec-rg \
  -l eastus \
  --sku standard


记住 Key Vault 名称，例如：

devops-kv-12345

🛠️ Part 2：向 Key Vault 添加 Secrets
az keyvault secret set \
  --vault-name devops-kv-12345 \
  --name api-token \
  --value "my-super-secret-token"


再添加一个数据库密码：

az keyvault secret set \
  --vault-name devops-kv-12345 \
  --name db-password \
  --value "P@ssw0rd!"


现在你有两个密钥：

api-token
db-password

🛠️ Part 3：创建 Azure DevOps Variable Group（Library）

进入：

Azure DevOps → Pipelines → Library → Variable Groups → + Variable Group

填写：

字段	值
Name	prod-secrets
Description	Production KeyVault-backed secrets

然后点击：

Link secrets from an Azure Key Vault
选择：

Service connection（若没有 → 创建 Azure Resource Manager 类型）

Subscription

Key Vault：devops-kv-12345

“Authorize” 按钮

勾选要导入的 secret：

api-token

db-password

保存后你会看到：

api-token = (from Key Vault)
db-password = (from Key Vault)


这些值不会暴露。

🛠️ Part 4：在 YAML Pipeline 中引用 Variable Group

在 CI/CD 的 YAML 最上面加入：

variables:
- group: prod-secrets


这样就能在 Pipeline 中读取：

$(api-token)
$(db-password)

🛠️ Part 5：在 Pipeline 中安全使用 Secret

示例：Node.js 项目

steps:
- script: |
    echo "Deploying with API token:"
    echo "$(api-token)"
  displayName: "Use API Token"


⚠️ 默认情况下，Azure DevOps 会将 secret 打码：

输出：

Deploying with API token:
***


确保不会泄露到日志。

🛠️ Part 6：在 Python 或 Node.js 项目中实际使用

Node.js：

- script: |
    export API_TOKEN=$(api-token)
    node deploy.js


Python：

- script: |
    export DB_PASSWORD=$(db-password)
    python deploy.py


部署脚本可以这样读变量：

Node.js:

console.log(process.env.API_TOKEN);


Python:

import os
print(os.environ["DB_PASSWORD"])

🧪 六、你今天应该完成（验证成果）

✔ 成功创建 Azure Key Vault
✔ 成功写入 secret（API Key / DB 密码）
✔ DevOps Variable Group 成功引用 Key Vault
✔ Pipeline 引用了 Variable Group
✔ Secret 在 pipeline 中安全显示（***）
✔ 你的项目能读取 secret 并运行

你已经掌握企业 DevOps 的核心技能：
安全变量管理 + KeyVault 集成

🧠 七、企业最佳实践（非常重要）
✔ 不在 Pipeline 中写明文密码
✔ 禁止在 Git Repo 中包含 config.json / secrets.json
✔ 环境必须分离：
dev-secrets
qa-secrets
prod-secrets

✔ 不要在 pipeline 中 echo secret（即使被打码）
✔ Key Vault secret 必须启用 RBAC
✔ 所有 Service Connection 应使用托管身份（Managed Identity）
📘 今日总结

今天你完成了企业级 DevOps 最关键的能力之一：

理解环境变量 / Secret 的类型

学会使用 Azure DevOps Library

实现 KeyVault → DevOps → Pipeline 的闭环

学会在管道中安全注入密码或 token

掌握跨环境 secret 管理的最佳实践

从今天起，你的 CI/CD 不再依赖明文密码，而是完全基于 安全密钥管理。