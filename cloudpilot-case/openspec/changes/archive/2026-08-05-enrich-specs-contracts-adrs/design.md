# Design · enrich-specs-contracts-adrs

## Context

CloudPilot 的规范工件过去只有 proposal/design/tasks/specs 四类（04-openspec 形态）。SDD book-code 的实践表明：**契约（contracts/）与架构决策（adrs/）应当是与 specs 并列的常驻工件**——契约被 design 与 tasks 共同引用、单独维护；决策带七段式留痕、可回读。本 change 把它们引入 `openspec/` 工作区，并把 billing 的编号缺口补齐。

## Goals

- contracts/ 与 adrs/ 成为常驻工件（不随 change 归档而移动），半年后仍可回读原始决策动机。
- billing 4 个 Requirement 获得可被验收门禁引用的编号（B-1~B-4）。
- 全程通过 openspec CLI 校验与归档，验证 change 生命周期真实可用。

## Non-Goals

- 不重构 `04-openspec/`（Golden Set 只读）。
- 不实现 TypeScript 契约代码（那是 P5 的 `@cloudpilot/contracts`，见 05 §3；本 change 只沉淀设计级契约工件）。
- 不引入新的 DDD 不变量（BillingContext 在模型里无不变量表，见 ADR-005）。

## Decisions

### D1：contracts/ 与 adrs/ 放在 `openspec/` 顶层常驻

- 选项：A) `openspec/` 顶层常驻；B) change 目录内（随 archive 移动）；C) `cloudpilot-case/` 与 04-openspec 并列。
- 决策：A。
- 理由：CLI 只消费 `openspec/specs` 与 `openspec/changes`，顶层未知目录不影响 validate/doctor；契约与决策是常驻主基线而非一次变更；B 会让归档后的契约脱离主基线，C 会脱离 CLI 工作区。
- 影响：`openspec/contracts/`、`openspec/adrs/` 随仓库版本管理，不随 archive 移动。

### D2：billing 编号用 B-N 派生，不编造 IV-9

- 选项：A) 回溯 ddd-modeler 给 BillingContext 补不变量表；B) 从 design D1-D4 派生 B-N 编号；C) 维持无编号。
- 决策：B。
- 理由：回溯会改动 Golden Set（03-ddd-modeling.md），违反只读约束；且 config.yaml 规则「不新增 DDD 模型未提及的概念」——B-N 是派生而非新增，正是该规则的正解；C 无法让 billing 被门禁引用。
- 影响：编号体系「IV-N = DDD 聚合不变量、B-N = design 决策派生约束」，溯源规则写入 ADR-005；若后续 DDD 模型补 IV-9（FR-11 到期告警），B-N 按 ADR-005 规则归并。

### D3：delta 的 MODIFIED Requirement 标题与主基线完全同名

- 选项：A) 标题加 B-N 后缀（如「实时报价（B-1）」）；B) 标题同名，编号放正文与 Rationale。
- 决策：B。
- 理由：实测 `openspec archive` 的 merge 按 Requirement 名与主基线匹配（`billing MODIFIED failed for header ... not found`），标题不同名会导致归档中止；validate 层虽通过，但 merge 层强制同名。
- 影响：billing 主基线标题保持「实时报价」等原名，B-N 标注在正文与 Rationale；resource-request 的 IV-N 在标题是历史生成风格，本次不回溯。

## Risks & Trade-offs

- **编号位置不一致**：billing 的 B-N 在正文而非标题，与 resource-request 的 IV-N 标题风格不同——可接受，merge 约束优先；`openspec spec show billing` 仍可检索到编号文本。
- **contracts/ 与 adrs/ 不被 CLI 校验**：它们是 SDD 惯例工件，依赖人工 review 保证质量——这正是 SDD 95/5 原则的边界（规范内容靠人，生命周期靠工具）。
- **05/06 IV-N 冲突残留**：05 §2.1 的 IV-3/IV-5/IV-6 映射与 03 不一致，06 §2 已声明以 03 为准；本 change 只在 project.md 已知 gap 记录裁决，不修改 05。
