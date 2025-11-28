Day 12：Azure Policy 与 Cost Management（控制成本与治理）
模块	内容
主题	Azure Policy 与 Cost Management
学习目标	理解 Azure 治理体系、策略控制（Policy）、预算与成本告警；掌握如何用策略管理资源合规、用预算控制云成本。
实战任务	设置成本预算（预算告警）、创建必需标签策略、限制 VM 类型策略、查看成本分布图。
🧠 一、核心知识讲解
1. Azure 治理（Governance）三大支柱

Azure 的企业级治理由以下三部分构成：

Management Group → 订阅管理（组织结构）
Azure Policy → 合规、限制、强制规则
Cost Management → 成本分析、预算、成本控制


Azure Policy：强制或审计资源是否符合规范，例如：

必须有 Tag（owner, environment 等）

禁止创建某些区域资源

限制 VM SKU

Cost Management：帮助你分析成本、设置预算、自动触发告警。

2. 什么是 Azure Policy？

Azure Policy 是一套“规则系统”，能够自动确保你的资源符合公司或团队的标准。

Policy 作用：

限制（deny）不符合规范的创建行为；

自动修复（modify）不合规资源；

审计（audit）不符合要求的资源。

Policy 三要素：
组件	说明
Policy Definition（策略定义）	规则的本体
Policy Assignment（策略分配）	把规则应用到订阅/资源组
Initiative（策略集合）	多条策略的组合包
3. 什么是 Cost Management？

Azure Cost Management 让你可以：

分析成本：按订阅、资源组、标签分类；

创建预算：限制每月消耗；

自动告警：当消耗超过阈值时发邮件/Teams；

查看趋势预测：了解未来成本曲线。

⚙️ 二、实战任务
任务 1：创建成本预算（Budget）并设置成本告警

示例：设置每月预算 $30，超 80% 自动告警。

az consumption budget create \
  --name monthly-budget \
  --category cost \
  --amount 30 \
  --time-grain monthly \
  --start-date 2025-01-01 \
  --end-date 2026-01-01 \
  --notification '{"Operator": "GreaterThan", "Threshold": 80, "ContactEmails": ["your@mail.com"]}'


💡效果：

超过 $24（80%）时触发邮件告警

可扩展多个通知方式：Email、Webhook、Teams、Function

任务 2：创建强制标签策略（Tag Policy）

资源必须包含 environment 标签。

az policy definition create \
  --name "require-environment-tag" \
  --display-name "Require Environment Tag" \
  --description "Ensure that resources have an environment tag." \
  --rules '{
      "if": {
        "field": "tags[environment]",
        "exists": "false"
      },
      "then": {
        "effect": "deny"
      }
    }' \
  --mode All


分配到资源组：

az policy assignment create \
  --name "require-environment-tag-assignment" \
  --scope $(az group show -n demo-rg --query id -o tsv) \
  --policy "require-environment-tag"


验证：
尝试创建未加标签资源 —— 会被直接拒绝。

任务 3：限制 VM SKU（控制成本）

只允许便宜的 B 系列虚拟机：

az policy definition create \
  --name "allowed-vm-sku" \
  --display-name "Allowed VM Sizes" \
  --description "Restrict VM size to B-Series only." \
  --rules '{
      "if": {
        "field": "type",
        "equals": "Microsoft.Compute/virtualMachines"
      },
      "then": {
        "effect": "deny",
        "details": {
          "notIn": ["Standard_B1s", "Standard_B2s"]
        }
      }
    }' \
  --mode All


分配到资源组：

az policy assignment create \
  --name "allowed-vm-assignment" \
  --scope $(az group show -n demo-rg --query id -o tsv) \
  --policy "allowed-vm-sku"


验证：
尝试创建 DS、F 系列 VM → 直接阻止。

任务 4：查看成本分析与资源费用排名
az costmanagement query --type Usage


在 Portal 可视化操作：

路径：
Cost Management → Cost analysis → Group by：Resource / Location / Tag

推荐观察内容：

哪些 VM 花钱最多？

哪些区域成本更高？

哪些资源组是成本热点？

哪些标签负责哪些费用？

🧩 三、验证成果

你应能完成：

✔ 创建每月成本预算 + 邮件告警
✔ 创建必需标签策略
✔ 创建限制 VM 系列策略
✔ 查看订阅/资源组的成本分析
✔ 让不符合策略的资源被自动拒绝创建

🧠 四、延伸与思考
治理最佳实践

所有资源必须包含成本标签（owner、costCenter、environment）

禁止 Production 使用昂贵 SKU

禁止创建公网 IP（Policy：deny public IP）

强制启用诊断日志（审计安全事件）

成本优化建议（非常实用）

开启 VM 自动关机（开发环境）

使用 Spot VM（最大程度省钱）

存储冷层（Cool / Archive）

用 Azure Advisor 查看节省建议

删除僵尸资源：孤立 IP、未使用磁盘、闲置 VM

📘 今日总结

你掌握了 Azure Policy 的核心原理与使用方法；

用策略限制资源部署，强制合规；

学会了成本预算、告警与成本分析；

能够主动控制开销，建立企业级治理体系；

完成了 Azure 企业治理三大核心之一：Cost + Policy + Compliance。