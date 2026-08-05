# ADR-005：IV-N / B-N 编号溯源规则（元决策）

## 状态

已采纳

## 背景

CloudPilot 的规范使用编号引用不变量：`resource-request` / `resource-management` 的 Requirement 标题挂 IV-N，而 `billing` 的 4 个 Requirement 无编号。SDD 的主张是「每个约束都要可被门禁精确引用」，因此需要裁决 billing 的编号体系，并固化编号溯源规则防止未来编造编号。

## 选项

| 选项 | 优点 | 缺点 |
|------|------|------|
| 回溯 ddd-modeler 给 BillingContext 补不变量表，挂 IV-9~IV-12 | 编号体系统一（全 IV-N） | 改动 `03-ddd-modeling.md`（评估基准 Golden Set，CLAUDE.md 禁止）；违背「不新增 DDD 模型未提及的概念」 |
| 从 design 关键决策 D1-D4 派生可测约束，编号 B-1~B-4 | 不碰 Golden Set；约束可测、可溯源；B 前缀区分来源 | 双编号体系，需文档解释 |
| 维持无编号 | 零工作 | 无法被验收门禁引用；与其余 spec 不一致 |

## 决策

billing 使用 **B-N 派生编号**（B-1~B-4，从 design.md D1-D4 派生可测约束）；编号溯源规则固化如下：

1. **IV-N**：唯一来源是 DDD 模型 `03-ddd-modeling.md` §III 聚合不变量表（IV-1~IV-8）。禁止凭空新增 IV 编号。
2. **B-N**：从 `04-openspec/design.md` 关键决策 D1-Dn 派生的可测约束，用于无 DDD 不变量表的子域（billing）。正文标注 + Rationale 写明派生来源。
3. **归并规则**：若后续 DDD 模型为 billing 补不变量表（如 FR-11 到期告警的 IV-9），B-N 按「同语义合并进 IV-N、无 IV 对应则保留 B-N」处理。
4. **冲突裁决**：编号冲突以 03 模型为准（06 §2 已声明 05 §2.1 的 IV-3/IV-5/IV-6 勘误）。

## 理由

1. `config.yaml` rules：specs「不得新增 DDD 模型未提及的概念；如有缺口，先回溯 ddd-modeler」——本案例回溯会破坏 Golden Set，派生是约束的正解
2. CLAUDE.md 约定：`04-openspec/`、`03-ddd-modeling.md` 是评估基准，`对比` 命令依赖，不可覆盖
3. billing 的 4 个 Requirement 均源自 D1-D4：实时报价（D2/D4）、快照（D2）、成本汇总（D3）、ACL（D1 + context-map）——派生来源可逐一引用
4. SDD book-code 实践：编号/决策留痕（ADR）让「为什么这么定」半年后可回读

## 影响

- 新增：`openspec/adrs/005-ivn-provenance.md`（本文件）
- 主基线变更：billing 4 个 Requirement 补 B-1~B-4（本 change）
- `project.md` 已知 gap 清单更新：billing 编号缺口由本 change 关闭

---

*ADR 版本：v1.0 | 本 change 元决策*
