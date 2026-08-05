# Change: enrich-specs-contracts-adrs

## Why

SDD（规范驱动开发）的核心主张是「规范是第一手工件」：契约与关键决策也应显式化，而不是只存在于设计文档的自由文本里。本工作区（`openspec/`）迁移自 `04-openspec/` 后仍有三处缺口：

1. **契约只存在于代码示例**：`05-p5-code-bridge.md` §3 的 `@cloudpilot/contracts` 契约包（状态枚举、VALID_TRANSITIONS、6 个领域事件、Provisioner 接口）没有独立规范工件，design 与 tasks 无法共同引用。
2. **关键决策无留痕**：design.md 的 D1-D4 是架构选型结论，但「为什么选、否决了什么、影响哪些文件」没有 ADR 记录，半年后无人能回答当初的取舍。
3. **billing 规范无编号**：billing 的 4 个 Requirement 未挂不变量编号（IV-N 或派生编号），与 resource-request / resource-management 的编号密度不一致，无法被验收门禁精确引用。

本 change 通过移植 SDD book-code 的 `contracts/` 与 `adrs/` 工件形态补上这三处，并建立 billing 的 B-N 派生编号体系（溯源规则见 ADR-005）。

## What Changes

1. 新增 `openspec/contracts/data-models.md`：从 05 §3 提取契约设计（状态枚举、VALID_TRANSITIONS 矩阵、6 个领域事件、Provisioner 接口、Quote/CostRecord/PricingTable、Repository 接口），作为常驻工件被 design 与 tasks 共同引用。
2. 新增 `openspec/adrs/001-005`：把 design.md D1-D4 提升为七段式 ADR（状态/背景/选项/决策/理由/影响），另加 005 记录 IV-N/B-N 编号溯源规则。
3. MODIFIED `billing` capability：4 个 Requirement 补 B-1~B-4 派生编号（正文标注 + Rationale 说明来源），不新增也不删除 Requirement。

## Out of scope

- 不改动 `04-openspec/`、`03-ddd-modeling.md`、`05-p5-code-bridge.md`、`06-p5-implementation-workflow.md`（评估基准 Golden Set，只读）。
- 不新增 DDD 模型未提及的领域概念；不编造 IV-9 编号。
- 不修正 05 §2.1 的 IV-N 编号冲突（裁决以 03 为准，已由 06 §2 声明）。
- 不写代码：本 change 只做规范工件，P5 代码实现不在范围内。

## Capabilities

- Modified: `billing`

## Impact

- 主基线 `openspec/specs/billing/spec.md` 的 4 个 Requirement 被 MODIFIED（正文标注 B-N + Rationale 补充）。
- 新增常驻目录 `openspec/contracts/` 与 `openspec/adrs/`（不进 change，不随 archive 移动）。
- 无代码变更、无数据迁移、无部署影响；回滚 = 从 git 恢复主基线文件。
