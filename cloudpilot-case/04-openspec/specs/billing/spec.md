# Spec · billing

> **Capability**：报价计算与项目成本汇总（Supporting 子域）
> **上游来源**：[`../../../03-ddd-modeling.md`](../../../03-ddd-modeling.md) §II 战略 `@ddd-contexts` BillingContext + §III 战术 `QuoteCalculator`
> **责任工件**：`PricingTable`、`Quote` (VO)、`CostRecord`

---

## ADDED Requirements

### Requirement: 实时报价

UI SHALL 在用户选择 `(type, spec, days)` 后于 200ms 内返回 `Quote`。

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

`ResourceRequest` 创建时 SHALL 复制当时 `Quote.totalPrice` 为不可变 `cost`。

#### Scenario: 提交后报价表变更不影响已存申请

- **GIVEN** `ResourceRequest R-001` 创建时 `cost = 700`
- **WHEN** 运维更新 `PricingTable[gpu-a100] = 120 元/天`
- **THEN** R-001 的 `cost` 仍为 700（IV-5）

---

### Requirement: 项目级成本汇总

系统 SHALL 按 `projectId` 实时汇总已发生与预计成本。

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

`ResourceRequest` 上下文 SHALL NOT 直接持有 `PricingTable` 引用，仅通过 `QuoteCalculator` 域服务接口交互。

#### Scenario: PricingTable schema 变更不污染 ResourceRequest

- **GIVEN** `PricingTable` 新增 `region` 字段
- **WHEN** ResourceRequest 上下文代码不变
- **THEN** 编译通过 / 测试通过

---

## Events

| 事件                | 触发条件        | 载荷                                   |
| :------------------ | :-------------- | :------------------------------------- |
| `QuoteCalculated`   | UI 调用计价     | type, spec, days, totalPrice           |
| `CostRecordUpdated` | 已发生/预计变更 | projectId, actual, forecast, updatedAt |

## Pricing Table 配置示例

```yaml
# pricing.yaml（Ops 维护）
- type: gpu-a100
  spec: 1x80GB
  unitPricePerDay: 100
- type: cpu-large
  spec: 16C64G
  unitPricePerDay: 30
- type: storage-ssd
  spec: 1TB
  unitPricePerDay: 5
```

## Repository

```typescript
interface PricingTable {
  lookup(type: ResourceType, spec: ResourceSpec): UnitPrice | null;
}

interface CostRecordRepository {
  findByProject(projectId: ProjectId): Promise<CostRecord>;
  save(record: CostRecord): Promise<void>;
}
```
