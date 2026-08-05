# OpenSpec 项目上下文 · CloudPilot

> 本文件是 `/opsx:*` 与 `openspec` CLI 每次执行时必读的项目上下文。**写得越具体，AI 跑偏概率越低**（移植自 [sdd-in-action 行动营 Week 2](https://github.com/huangjia2019/sdd-in-action) 的 project.md 写法）。

## 项目

**CloudPilot 云管平台 MVP** —— AI Native DevOps 8 阶段框架（P1-pre → P7）案例的 OpenSpec CLI 实况工作区。主基线规范迁移自 `04-openspec/`（评估基准，只读），本工作区是其可验证、可归档的实况副本。

## 技术栈

- 契约设计：TypeScript（`@cloudpilot/contracts`，见 `05-p5-code-bridge.md` §3，尚未实现）
- Mock UI：单文件 HTML + localStorage + `setInterval` 5s 模拟异步配置
- 真实云 SDK：阿里云 ECS/RDS/OSS/Redis/SLB（P7 演进，接口预留 `Provisioner`）
- 本工作区只承载规范工件（specs/contracts/adrs/changes），不承载代码

## 关键数据契约

- 状态枚举 `ResourceRequestStatus`：`PENDING / APPROVED / REJECTED / PROVISIONED / RELEASED`
- 状态转换矩阵 `VALID_TRANSITIONS`（↔ IV-2 合法路径、IV-3 终态、IV-8 释放）
- 6 个领域事件：`ResourceRequested / RequestApproved / RequestRejected / ResourceProvisionRequested / ResourceProvisioned / ResourceReleased`
- `Quote`（值对象，提交时快照进 `ResourceRequest.cost`，不持久化）、`CostRecord`、`PricingTable`
- `Provisioner` 接口（`provision` / `release`）+ `MockProvisioner` / `AliyunProvisioner`
- `ResourceRequestRepository` 接口

## 项目约定

- 不变量编号 `IV-N` 唯一来源是 DDD 模型 `03-ddd-modeling.md` §III（IV-1~IV-8）；**禁止凭空新增 IV 编号**
- billing 用 `B-N` 派生编号（从 design 关键决策 D1-D4 派生可测约束），见 ADR-005
- 规范语言只用 SHALL / MUST（禁 SHOULD / MAY）；Requirement 标题 `### Requirement: <描述>（IV-N / B-N）`
- capability 命名 kebab-case；领域事件 PastTenseVerb；Repository 接口 `<Aggregate>Repository`
- 跨上下文仅通过领域事件（最终一致，< 30s 容忍）+ ACL 防腐层；不引入分布式事务

## 已知 gap（由 changes 逐步修复）

1. **billing 4 个 Requirement 无编号**（实时报价 / 报价快照 / 项目级成本汇总 / ACL 防腐层）——BillingContext 在 DDD 模型里无不变量表，计划用 B-1~B-4 派生编号
2. **缺 contracts/ 与 adrs/** —— sdd-paradigms-synthesis.md §1.2 指出的 P4 显式化缺口；契约只存在于 05 §3 的代码示例中
3. **05-p5-code-bridge.md §2.1 的 IV-3/IV-5/IV-6 映射与 DDD 模型不一致** —— 以 `03-ddd-modeling.md` 编号为准（06 §2 已声明勘误）；05 是评估基准不可改，只记录裁决
4. **CLI 生命周期（onboard → change → archive）从未真实走完** —— Stage E 归档在案例重放中被跳过（本工作区已走通，见 changes/archive/）
5. **04-openspec 既有 FR 引用错误**（Golden Set 只读，留待后续 change 修正）：billing spec「UI 禁用提交按钮（FR-08）」的 FR-08 实为项目成本中心、与禁用提交无关；04-openspec/tasks.md「实时报价调用 QuoteCalculator（FR-03）」应为 FR-02（实时报价 <500ms）

## 术语表（避免 AI 理解偏差）

| 词 | 本工作区里指 |
|:---|:---------|
| 04-openspec | **评估基准（Golden Set）**，只读；本工作区是副本，二者可 diff 对照、互不覆盖 |
| change | `openspec/changes/<name>/` 下的一次待合并变更（proposal/design/tasks/specs delta） |
| delta | change 内 `specs/<capability>/spec.md` 的增量（ADDED / MODIFIED / REMOVED Requirements） |
| capability | OpenSpec 能力单元（`resource-request` / `resource-management` / `billing`） |
| B-N | 从 design 关键决策派生的可测约束编号（B-1~B-4），与 DDD 聚合不变量 IV-N 区分 |
| contracts/ | `openspec/contracts/` 常驻契约工件（被 design 与 tasks 共同引用，SDD book-code 惯例） |
| adrs/ | `openspec/adrs/` 常驻架构决策记录（七段式，SDD book-code 惯例） |
