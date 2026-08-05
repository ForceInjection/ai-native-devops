# Tasks · enrich-specs-contracts-adrs

## Phase 1: 契约提取

- [x] 从 `05-p5-code-bridge.md` §3 提取契约设计 → `openspec/contracts/data-models.md`（状态枚举、VALID_TRANSITIONS、6 个领域事件、Provisioner 接口、Quote/CostRecord/PricingTable、Repository 接口，来源标注 03 §III + 05 §3）

## Phase 2: ADR 编写

- [x] `openspec/adrs/001-mock-provisioner-interface.md`（D1 → 七段式）
- [x] `openspec/adrs/002-quote-snapshot-not-persisted.md`（D2 → 七段式）
- [x] `openspec/adrs/003-event-driven-final-consistency.md`（D3 → 七段式）
- [x] `openspec/adrs/004-localstorage-setinterval-mvp.md`（D4 → 七段式）
- [x] `openspec/adrs/005-ivn-provenance.md`（IV-N/B-N 编号溯源规则）

## Phase 3: billing delta

- [x] `specs/billing/spec.md`：4 个 Requirement 走 MODIFIED（标题同名、正文标 B-1~B-4、补 Rationale 来源）

## Phase 4: 格式对齐与验证

- [x] `openspec validate enrich-specs-contracts-adrs` 通过（0 errors）

## Phase 5: 归档与验收

- [x] `openspec archive enrich-specs-contracts-adrs --yes`：主基线 billing 合并（Totals: + 0, ~ 4, - 0）、change 移入 `changes/archive/2026-08-05-enrich-specs-contracts-adrs/`
- [x] `openspec validate --all`（3 passed, 0 failed）+ `openspec doctor`（root ok）终态校验
