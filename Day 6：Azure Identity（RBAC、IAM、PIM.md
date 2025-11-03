Day 6：Azure Identity（RBAC、IAM、PIM）
模块	内容
主题	Azure 身份与访问控制（RBAC、IAM、PIM）
学习目标	理解 Azure 的身份管理模型，掌握基于角色的访问控制（RBAC）与权限分配方法，了解临时权限与合规机制（PIM）。
实战任务	创建用户与角色，在资源组上分配访问权限，使用 RBAC 控制访问范围。
🧠 一、核心知识讲解
1. Azure 身份体系的三层结构

Azure 的访问控制体系基于 Azure Active Directory（AAD / Entra ID），由三层构成：

     用户（User）/ 服务主体（Service Principal）
          ↓
     角色（Role）
          ↓
     作用域（Scope：Management Group / Subscription / Resource Group / Resource）


层级	示例	说明
用户 / 组 / 服务主体	zhi.hu@microsoft.com
, myApp-sp	发起操作的身份
角色 (Role)	Reader、Contributor、Owner、自定义角色	定义“能做什么”
作用域 (Scope)	Subscription → Resource Group → Resource	定义“能在哪里做”

💡核心逻辑：

权限 = 角色 + 作用域 + 身份

例如：

将 “Contributor” 角色授予用户 A，在 “learn-azure-rg” 资源组上。
→ 用户 A 能在该资源组内创建/删除资源，但无法访问其他组。

2. 常见内置角色
角色	权限	典型场景
Owner	拥有全部权限，包括分配权限	管理员
Contributor	创建/修改资源，但无法分配权限	开发者
Reader	仅查看资源	审计员
User Access Administrator	管理他人权限	安全管理员

Azure 还提供细粒度角色，如：

Storage Blob Data Reader

Virtual Machine Contributor

Key Vault Administrator

3. RBAC 与 IAM 的区别
项目	RBAC (Role-Based Access Control)	IAM (Identity & Access Management)
核心	通过角色授权	用户与身份生命周期管理
重点	“谁能访问什么”	“谁存在 + 如何认证 + 授权策略”
实现	分配角色	管理用户、组、服务主体
示例	给组 Developer 赋予 Contributor	为新员工创建账户并强制 MFA

总结：

IAM = 管理“身份”

RBAC = 管理“权限”

4. PIM（Privileged Identity Management）

PIM（特权身份管理）用于临时授权和最小权限控制，常用于企业合规。

用户不是一直拥有权限，而是在“需要时”申请激活；

可设置自动审批、时间限制、活动记录；

常用于“全局管理员”、“Owner”等高权限角色。

💡 典型应用场景：

“开发者需要临时访问生产环境 2 小时”
→ PIM 激活后自动授予权限，时间到期自动收回。

🧰 二、实战任务
任务 1：查看当前登录用户和订阅
az account show
az ad signed-in-user show

任务 2：列出内置角色
az role definition list --output table

任务 3：在 Azure AD 中创建一个用户

（需有管理员权限，否则可跳过并使用已有用户）

az ad user create \
  --display-name "Test User" \
  --user-principal-name testuser@yourtenant.onmicrosoft.com \
  --password "P@ssword123!"


验证：

az ad user list --output table

任务 4：将角色分配给用户

为该用户分配 Reader 角色，仅限 learn-azure-rg 资源组：

az role assignment create \
  --assignee testuser@yourtenant.onmicrosoft.com \
  --role "Reader" \
  --resource-group learn-azure-rg


验证：

az role assignment list \
  --assignee testuser@yourtenant.onmicrosoft.com \
  --output table

任务 5：使用自定义角色（可选）

创建一个自定义角色，只允许查看和重启虚拟机：

保存文件 custom-role.json：

{
  "Name": "VM Restart Operator",
  "Description": "Can view and restart virtual machines.",
  "Actions": [
    "Microsoft.Compute/virtualMachines/read",
    "Microsoft.Compute/virtualMachines/restart/action"
  ],
  "NotActions": [],
  "AssignableScopes": ["/subscriptions/<your-subscription-id>"]
}


创建角色：

az role definition create --role-definition custom-role.json


分配：

az role assignment create \
  --assignee testuser@yourtenant.onmicrosoft.com \
  --role "VM Restart Operator"

任务 6：（了解）PIM 激活示例（仅可视化操作）

在 Portal 中：

搜索 “Azure AD Privileged Identity Management”；

打开“我的角色 (My roles)”；

激活或请求特权角色；

查看活动记录与自动到期时间。

🧩 三、验证成果

✅ 你应能完成：

成功创建用户或识别已有账户；

在资源组级别分配角色；

验证权限是否生效（可登录验证能否修改资源）；

理解角色、范围、权限三要素；

理解 PIM 的工作原理与使用场景。

🧠 四、延伸与思考

最小权限原则（Least Privilege Principle）

永远授予最小必要权限；

避免“Owner”滥用；

定期审查角色分配。

多层授权设计

订阅级：全局管理；

资源组级：项目管理；

资源级：精细控制。

企业合规实践

启用 MFA（多因素认证）；

配置 Conditional Access；

使用 PIM 管理临时高权限。

推荐阅读

Azure RBAC 官方文档

Azure AD Privileged Identity Management

RBAC 最佳实践

📘 今日总结

你理解了 Azure 的身份与权限控制体系；

学会了 RBAC 的角色、范围、用户三要素；

实操了角色分配与权限验证；

了解了 PIM 的临时授权机制；

掌握了企业级安全管理的基础知识。
