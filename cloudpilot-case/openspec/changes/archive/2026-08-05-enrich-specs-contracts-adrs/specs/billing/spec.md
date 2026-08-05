# billing Specification

## MODIFIED Requirements

### Requirement: 实时报价

UI SHALL 在用户选择 `(type, spec, days)` 后于 500ms 内返回 `Quote`（B-1，派生自 design.md D2/D4，溯源见 ADR-005）。

**Priority**: P0 (Critical)

**Rationale**: 报价是提交前置动作（FR-02 / NFR-01）；B-1 从 design.md D2/D4 与 FR-02 派生，原 200ms 目标放宽为 500ms 上限承诺（对齐 PRD 的 <500ms 预算）。

#### Scenario: 选择 GPU-A100 / 7 天

- **GIVEN** `PricingTable[gpu-a100] = 100 元/天`
- **WHEN** UI 调用 `QuoteCalculator.calc({ type: 'gpu-a100', days: 7 })`
- **THEN** 返回 `Quote { unitPrice: 100, days: 7, totalPrice: 700, calculatedAt }`

#### Scenario: 未知规格返回错误

- **GIVEN** `PricingTable` 中无 `(type, spec)` 项
- **WHEN** 调用 `QuoteCalculator.calc`
- **THEN** 抛 `UnknownPricingItem`
- **AND** UI 禁用提交按钮（FR-08）

---

### Requirement: 报价快照写入申请单

`ResourceRequest` 创建时 SHALL 复制当时 `Quote.totalPrice` 为不可变 `cost`（B-2，派生自 design.md D2，与 IV-5 联动，溯源见 ADR-005）。

**Priority**: P0 (Critical)

**Rationale**: 关键决策 D2（Quote 作为创建快照）；B-2 是 D2 的可测化，与 resource-request 的 IV-5 成本不可变共同构成「快照隔离调价」约束。

#### Scenario: 提交后报价表变更不影响已存申请

- **GIVEN** `ResourceRequest R-001` 创建时 `cost = 700`
- **WHEN** 运维更新 `PricingTable[gpu-a100] = 120 元/天`
- **THEN** R-001 的 `cost` 仍为 700（IV-5）

---

### Requirement: 项目级成本汇总

系统 SHALL 按 `projectId` 实时汇总已发生与预计成本（B-3，派生自 design.md D3，溯源见 ADR-005）。

**Priority**: P1 (Important)

**Rationale**: 关键决策 D3（跨上下文领域事件 + 最终一致）；B-3 要求成本读模型由领域事件驱动更新，可测性依赖 `ResourceProvisioned` / `ResourceReleased` 事件流。

#### Scenario: 配置完成累加已发生

- **GIVEN** `CostRecord[project-X].actual = 0`
- **WHEN** 收到 `ResourceProvisioned(R-001)` 事件，R-001 项目为 X，cost = 700
- **THEN** `CostRecord[project-X].actual += 按配置至今的天数 × 单价`

#### Scenario: 释放后停止累加

- **GIVEN** `R-001` 已配置在项目 X
- **WHEN** 收到 `ResourceReleased(R-001)`
- **THEN** 停止累加，最终值锁定到 `actual`

#### Scenario: 预计成本

- **GIVEN** `R-001 status = PROVISIONED`，剩余天数 = 5
- **WHEN** 计算 `CostRecord[project-X].forecast`
- **THEN** 等于所有未释放申请的 `unitPrice × 剩余天数` 之和

---

### Requirement: ACL 防腐层

`ResourceRequest` 上下文 SHALL NOT 直接持有 `PricingTable` 引用，仅通过 `QuoteCalculator` 域服务接口交互（B-4，派生自 design.md D1 与 context-map ACL 模式，溯源见 ADR-005）。

**Priority**: P1 (Important)

**Rationale**: context-map ACL 集成模式；B-4 是 D1 接口对齐在边界层的可测化——PricingTable 的 schema 变更不得污染 ResourceRequest 上下文。

#### Scenario: PricingTable schema 变更不污染 ResourceRequest

- **GIVEN** `PricingTable` 新增 `region` 字段
- **WHEN** ResourceRequest 上下文代码不变
- **THEN** 编译通过 / 测试通过
