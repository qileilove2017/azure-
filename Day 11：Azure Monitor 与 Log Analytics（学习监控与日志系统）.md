Day 11：Azure Monitor 与 Log Analytics（学习监控与日志系统）
模块	内容
主题	Azure Monitor、Log Analytics
学习目标	理解 Azure 监控体系；掌握指标收集、日志分析与告警配置；学会构建自定义 Dashboard 展示虚拟机状态。
实战任务	创建 Log Analytics 工作区并连接 VM，编写 KQL 查询分析性能指标，创建 Azure Dashboard 展示实时运行状态。
🧠 一、核心知识讲解
1. 什么是 Azure Monitor？

Azure Monitor 是 Azure 的统一监控平台，
用于采集、分析并可视化所有资源的运行数据。

核心组成模块：

Azure Monitor
 ├── Metrics（性能指标：CPU、内存、磁盘）
 ├── Logs（操作/系统日志，KQL 查询）
 ├── Alerts（告警与自动响应）
 ├── Insights（应用、VM、容器分析）
 ├── Workbooks（交互式报告）
 └── Dashboard（可视化监控中心）


💡 一句话总结：

Azure Monitor 是连接“数据收集 → 分析 → 告警 → 可视化”的核心中枢。

2. Metrics vs Logs 的区别
对比项	Metrics（指标）	Logs（日志）
数据类型	数值型（CPU%、延迟）	结构化文本
收集频率	秒级	事件驱动或分钟级
查询工具	Metrics Explorer	Log Analytics（KQL）
典型用途	性能趋势分析	故障排查、根因分析
3. Log Analytics Workspace（日志分析工作区）

这是 Azure Monitor 的核心数据库，
所有 VM、App、Network 的日志数据都会汇聚到这里。

项目	内容
功能	存储、查询、分析日志
查询语言	Kusto Query Language (KQL)
常见来源	VM Agent、Azure Diagnostic、Application Insights
4. KQL（Kusto Query Language）简介

KQL 是 Azure 日志查询语言，
用于从 Log Analytics 中提取和分析监控数据。

示例：查询过去1小时内 VM 心跳次数

Heartbeat
| where TimeGenerated > ago(1h)
| summarize count() by Computer


查询过去1小时内 CPU 使用率：

Perf
| where ObjectName == "Processor" and CounterName == "% Processor Time"
| summarize AvgCPU = avg(CounterValue) by bin(TimeGenerated, 5m), Computer
| render timechart

⚙️ 二、实战任务
任务 1：创建 Log Analytics Workspace
az monitor log-analytics workspace create \
  --resource-group monitor-lab-rg \
  --workspace-name vm-monitor-ws \
  --location eastus


验证：

az monitor log-analytics workspace list --output table

任务 2：连接虚拟机到工作区

假设已有虚拟机 demo-vm：

az monitor log-analytics workspace get-shared-keys \
  --resource-group monitor-lab-rg \
  --workspace-name vm-monitor-ws


安装监控代理扩展：

az vm extension set \
  --publisher Microsoft.EnterpriseCloud.Monitoring \
  --name OmsAgentForLinux \
  --resource-group monitor-lab-rg \
  --vm-name demo-vm \
  --settings '{"workspaceId": "<workspace-id>"}' \
  --protected-settings '{"workspaceKey": "<primary-key>"}'


💡 完成后，几分钟内会看到日志流入 Log Analytics。

任务 3：在 Portal 中验证数据

进入：

Azure Portal → Log Analytics 工作区 → Logs

输入 KQL 查询：

Heartbeat
| where TimeGenerated > ago(30m)
| summarize count() by Computer


✅ 若返回结果中出现你的 VM 名称，表示连接成功。

任务 4：查看 VM Metrics（指标监控）

在 Portal：

Azure Monitor → Metrics → 选择目标 VM

选择指标：

CPU Percentage

Disk IOPS

Network In/Out

可设置：

时间范围：过去 1 小时

聚合方式：平均值

可视化：折线图 / 面积图

任务 5：创建告警规则（Alert）

打开 Portal → Azure Monitor → Alerts

选择 “New alert rule”

目标 (Resource)： demo-vm

信号 (Signal type)： Percentage CPU

条件： CPU > 80% 持续 5 分钟

Action Group： 发送邮件 / Teams 通知

保存并启用。

测试方法（在 VM 内运行高负载）：

yes > /dev/null &


几分钟后你将收到告警通知。

任务 6：创建 Azure Dashboard 展示 VM 状态

打开 Portal → Dashboard → “+ New Dashboard”

点击 “Edit” → “Add a resource tile”

添加以下组件：

Metrics 图表：显示 CPU 使用率；

Log Analytics 查询图表；

Resource Health（资源健康）；

Alert Summary（告警摘要）；

Cost Analysis（费用概览）。

保存为：

VM-Monitoring-Dashboard


💡 高级技巧：
Dashboard 支持 JSON 导出，可通过 Git 版本化管理。

任务 7：（可选）用 Workbook 制作交互式报告

打开 Azure Monitor → Workbooks → “+ New”

添加查询：

Perf
| where ObjectName == "Processor" and CounterName == "% Processor Time"
| summarize AvgCPU = avg(CounterValue) by Computer


添加图表组件（折线 / 柱状）；

添加参数过滤器（时间范围 / VM 名称）；

保存为 “CPU Performance Report”。

🧩 三、验证成果

✅ 你应能完成：

创建 Log Analytics Workspace；

将 VM 接入监控系统；

执行基本 KQL 查询；

创建告警规则与自动邮件通知；

搭建实时监控 Dashboard 展示 VM 状态。

🧠 四、延伸与思考

可观测性三要素

Metrics（指标）：资源性能；

Logs（日志）：事件记录；

Traces（调用链）：请求路径。

整合方案

对容器化环境：使用 Azure Monitor for Containers；

对应用层：接入 Application Insights；

对网络层：启用 Network Watcher + Flow Log；

对安全层：启用 Microsoft Defender for Cloud。

仪表盘设计建议

左上：系统健康 (VM / Storage / Network)

右上：资源使用率 (CPU、内存、IO)

左下：告警摘要

右下：趋势图 / 成本曲线

📘 今日总结

你理解了 Azure Monitor 的整体架构；

掌握了 Log Analytics 与 KQL 查询；

为 VM 建立了性能监控与日志分析体系；

创建了 Dashboard，可视化展示系统运行状态；

建立了告警机制，为后续自动化运维打下基础。