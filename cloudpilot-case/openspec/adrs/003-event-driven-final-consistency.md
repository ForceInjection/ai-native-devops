# ADR-003：跨上下文一致性用领域事件 + 最终一致，不引入分布式事务

## 状态

已采纳

## 背景

`ResourceRequest` 状态推进依赖 `ResourceManagement` 配置完成（`APPROVED → PROVISIONED`），且 `Billing` 的成本汇总依赖配置/释放事件。跨上下文一致性策略需要裁决。

## 选项

| 选项 | 优点 | 缺点 |
|------|------|------|
| 分布式事务（两阶段提交） | 强一致，状态立即同步 | 云原生环境引入协调器复杂度；`billing` 是 Supporting 子域不值得 |
| 领域事件 + 最终一致（Pub/Sub） | 无协调器；事件可重放、可审计；上下游解耦 | 状态短暂不一致（< 30s 可接受）；需幂等处理（IV-7） |
| 同步 RPC 直接推进 | 简单直接 | 上游依赖下游可用性；配置失败链路复杂化 |

## 决策

`ResourceProvisioned` 事件回流，`ResourceRequest` 监听并推进 `APPROVED → PROVISIONED`；成本读模型由 `ResourceProvisioned` / `ResourceReleased` 事件驱动。状态短暂不一致（< 30s）可接受，不引入分布式事务。

## 理由

1. `design.md` D3（04-openspec）：「状态短暂不一致（< 30s）可接受；不引入分布式事务」
2. `billing` spec「项目级成本汇总」（B-3）：收到 `ResourceProvisioned` / `ResourceReleased` 事件更新 `CostRecord`——读模型的事件驱动语义
3. `resource-request` spec「状态转换合法路径（IV-2）」：`ResourceProvisioned(requestId)` 事件监听推进——事件即状态机的输入
4. `resource-management` spec「资源实例唯一性（IV-7）」：事件重放去重——最终一致的前提能力已具备
5. `design.md` 集成与契约表：`Pub/Sub on ResourceProvisionRequested`，契约所有权归上游 OHS

## 影响

- 新增：`contracts/data-models.md` §2 的 6 个领域事件（`ResourceProvisionRequested` / `ResourceProvisioned` / `ResourceReleased` 等）
- 主基线变更：billing「项目级成本汇总」补 B-3 编号（本 change）
- 无代码影响：P5 实现时事件总线为 Mock（`setInterval` 模拟，见 ADR-004）

---

*ADR 版本：v1.0 | 来源：design.md D3（04-openspec）*
