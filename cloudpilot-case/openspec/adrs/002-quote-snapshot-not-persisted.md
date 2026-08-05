# ADR-002：Quote 不持久化，提交时快照进 ResourceRequest.cost

## 状态

已采纳

## 背景

报价（`Quote`）在提交申请前展示给用户，而 `ResourceRequest.cost` 是聚合根上的不可变成本。需要裁决：`Quote` 是否是独立聚合、是否持久化。

## 选项

| 选项 | 优点 | 缺点 |
|------|------|------|
| `Quote` 作为独立聚合持久化 | 历史报价可审计；可做报价版本管理 | 引入 Quote 失效语义（报价过期、版本比对）；聚合增多 |
| `Quote` 是值对象，提交时复制为 `ResourceRequest.cost`，原 Quote 不存 | 无失效语义；成本随申请单走，天然满足 IV-5 | 无法回溯「提交时看到的具体报价」（成本已固化，可接受） |
| 值对象 + 审计表 | 兼具前两者 | MVP 复杂度超标（YAGNI） |

## 决策

`Quote` 是值对象，提交时复制为 `ResourceRequest.cost`，原 Quote 不存。后续若需历史报价审计再升级。

## 理由

1. `design.md` D2（04-openspec）：「Quote 是值对象，提交时复制为 ResourceRequest.cost，原 Quote 不存」
2. `billing` spec「报价快照写入申请单」（B-2）：创建时 SHALL 复制当时 `Quote.totalPrice` 为不可变 `cost`——快照与调价隔离（IV-5）
3. `resource-request` spec「成本不可变（IV-5）」：`cost` 创建时确定、生命周期内不可变更——值对象方案与 IV-5 天然一致
4. 若 Quote 持久化，IV-5 之外还要管「报价表版本」约束，超出 Supporting 子域（`billing`）的必要复杂度

## 影响

- 新增：`contracts/data-models.md` §4 的 `Quote` 值对象与 `ResourceRequest.cost` 不可变快照
- 主基线变更：billing「报价快照写入申请单」补 B-2 编号（本 change）
- 无代码影响：P5 实现时 `QuoteCalculator` 返回值直接写入聚合命令

---

*ADR 版本：v1.0 | 来源：design.md D2（04-openspec）*
