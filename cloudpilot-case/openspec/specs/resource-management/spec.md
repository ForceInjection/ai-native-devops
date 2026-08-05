# resource-management Specification

> **Capability**：资源生命周期管理（Core 子域，MVP Mock 实现）
> **上游来源**：[`../../../03-ddd-modeling.md`](../../../03-ddd-modeling.md) §III 战术 `@ddd-aggregates` 聚合 2（IV-7, IV-8）+ `@ddd-domain-interactions` Provisioner 接口
> **责任聚合**：`ResourceInstance`

## Purpose

定义资源生命周期管理（Core 子域）的规范基线：实例唯一性、释放终态、Provisioner 接口契约与配置失败保留状态，来源于 DDD 模型聚合 2 与领域服务接口，供 P5 代码桥接与测试映射使用。

## Requirements

### Requirement: 资源实例唯一性（IV-7）

每个 `requestId` SHALL 至多对应一个 `ResourceInstance`。

**Priority**: P0 (Critical)

**Rationale**: 不变量 IV-7；防事件重放产生重复实例。

#### Scenario: 重复配置请求被去重

- **GIVEN** `ResourceInstance` 已存在 with `requestId = R-001`
- **WHEN** 重复收到 `ResourceProvisionRequested(R-001)`
- **THEN** 不创建新实例，发布幂等成功事件 `ResourceProvisioned(R-001)`

---

### Requirement: 释放后不可重新激活（IV-8）

`ResourceInstance` 进入 `RELEASED` 后 SHALL NOT 变回任何活动状态。

**Priority**: P0 (Critical)

**Rationale**: 不变量 IV-8；释放是终态，与 IV-2 的 `PROVISIONED → RELEASED` 单向一致。

#### Scenario: 释放后再次配置被拒

- **GIVEN** `ResourceInstance.status = RELEASED`
- **WHEN** 收到针对同 `requestId` 的 `ResourceProvisionRequested`
- **THEN** 抛出 `ResourceAlreadyReleased`

---

### Requirement: Provisioner 接口契约（D1）

所有 `Provisioner` 实现（Mock / Aws / Aliyun）SHALL 遵守同一接口契约。

**Priority**: P0 (Critical)

**Rationale**: 关键决策 D1（05 §3 契约包）；接口对齐真实云 SDK，契约测试覆盖 5 状态转换，详见 ADR-001。

#### Scenario: MockProvisioner 异步完成

- **GIVEN** `MockProvisioner` 收到 `provision(req)`
- **WHEN** 5 秒延迟后
- **THEN** 发布 `ResourceProvisioned(requestId, provisionedAt)` 事件

#### Scenario: 真实 SDK 实现通过同一契约测试

- **GIVEN** `AwsProvisioner` 实例
- **WHEN** 跑 `contracts/provisioner.contract.ts` 中所有 case
- **THEN** 全部通过

---

### Requirement: 配置失败保留申请状态

配置失败时，上游 `ResourceRequest` SHALL 保持 `APPROVED` 状态，不前进也不回滚。

**Priority**: P1 (Important)

**Rationale**: 集成失败模式（design.md 集成与契约表）；与 IV-3 超时告警链路联动。

#### Scenario: Provisioner 抛错

- **GIVEN** Provisioner 调用抛 `ProvisioningFailed`
- **WHEN** ResourceManagement 捕获
- **THEN** 不发 `ResourceProvisioned` 事件
- **AND** 发 `ProvisioningFailed(requestId, error)` 事件
- **AND** ResourceRequest 状态保持 `APPROVED`，触发 IV-3 超时告警链路

---

## Events

| 事件                  | 触发条件             | 载荷                                 |
| :-------------------- | :------------------- | :----------------------------------- |
| `ResourceProvisioned` | Provisioner 成功完成 | requestId, provisionedAt, instanceId |
| `ProvisioningFailed`  | Provisioner 抛错     | requestId, error, failedAt           |

## Repository

```typescript
interface ResourceInstanceRepository {
  findByRequestId(id: RequestId): Promise<ResourceInstance | null>;
  save(instance: ResourceInstance): Promise<void>;
}

interface Provisioner {
  provision(req: ProvisioningCommand): Promise<void>;
  release(requestId: RequestId): Promise<void>;
}
```
