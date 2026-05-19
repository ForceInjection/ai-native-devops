# Proposal · CloudPilot MVP

> **阶段**：AI-Native DevOps P4 OpenSpec 规范定义
> **上游输入**：[`../03-ddd-modeling.md`](../03-ddd-modeling.md) §II 战略（子域 + 上下文）
> **下游消费**：P5 代码生成（spec → code）、P6 评审、P7 部署
> **责任人**：架构师 + R-Lead 评审；AI 由 `@ddd-openspec-bridge` 直接生成草稿
> **AI 草稿置信度**：高（结构化映射 + 模型评审 100% 不变量覆盖）

---

## Why

研发团队当前申请云资源走 OA 工单 + 运维手工配置，**平均交付 2~3 天**；财务无项目级成本视图，无法做事前预算控制。详见 [`../01-interview-notes.md`](../01-interview-notes.md) 痛点 P1~P6 与 [`../02-prd.md`](../02-prd.md) §1。

**度量目标**：

| 指标             | 现状   | 目标         |
| :--------------- | :----- | :----------- |
| 申请→可用 中位   | 2~3 天 | ≤ 30 min     |
| 规格手填错误率   | ~15%   | 0%（结构化） |
| 项目级成本可见性 | 无     | 实时         |

## What Changes

引入 **CloudPilot** 平台，落地 3 个限界上下文：

- **ResourceRequestContext**（Core）：自助申请 + 团队内审批 + 5 状态机
- **ResourceManagementContext**（Core）：资源生命周期，MVP 用 Mock Provisioner，接口契约对齐真实云 SDK
- **BillingContext**（Supporting）：标准化 PricingTable + 项目级成本汇总

**MVP 范围**：4 个 must-have 功能（F1 自助申请 / F2 团队内审批 / F3 实时报价 / F4 项目成本视图）。F5 配额与 F6 到期告警纳入后续迭代。

## Impact

| 维度         | 影响                                                                                         |
| :----------- | :------------------------------------------------------------------------------------------- |
| **新增能力** | `resource-request`、`resource-management`、`billing` 3 个 spec capability                    |
| **依赖**     | 复用现有 SSO（鉴权）、IM 平台（通知）；新建 PricingTable 配置                                |
| **数据迁移** | 无（新建）                                                                                   |
| **回滚策略** | Mock Provisioner 可一键关闭恢复 OA 工单流；DB schema 独立                                    |
| **风险**     | R1 Mock 与真实云 SDK 行为分叉 → 契约测试缓解；R2 审批人离线阻塞 → MVP 不处理，迭代加超时升级 |

## Non-Goals

- 多云调度 / 跨云成本对比
- 财务结算与发票
- 跨 BU 复杂审批流（仅团队内单级审批）
- 实时计量（按天估算即可）
