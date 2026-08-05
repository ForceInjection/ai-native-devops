# Contracts · CloudPilot 数据模型与接口契约

> 常驻契约工件：被 `design.md` 与 `tasks.md` 共同引用、单独维护（SDD book-code 惯例）。
> **来源**：[`03-ddd-modeling.md`](../../03-ddd-modeling.md) §III 战术接口 + [`05-p5-code-bridge.md`](../../05-p5-code-bridge.md) §3 `@cloudpilot/contracts`。
> **权威性裁决**：本文件是接口字段的权威来源；`specs/*/spec.md` 的 `## Events` 表是面向场景的简化视图，字段以本文件为准。

## 1. 状态枚举与转换矩阵

`ResourceRequestStatus` 五个值之一（↔ IV-1 枚举受限）：

| 枚举值 | 语义 | 相关不变量 |
| :--- | :--- | :--- |
| `PENDING` | 已提交待审批 | IV-1 |
| `APPROVED` | 审批通过待配置 | IV-1, IV-2 |
| `REJECTED` | 已拒绝（终态） | IV-1, IV-2 |
| `PROVISIONED` | 已配置完成 | IV-1, IV-2 |
| `RELEASED` | 已释放（终态） | IV-1, IV-2, IV-8 |

`VALID_TRANSITIONS` 矩阵（直接编码 IV-2 合法转换路径）：

| 当前状态 | 可转换到 |
| :--- | :--- |
| `PENDING` | `APPROVED`, `REJECTED` |
| `APPROVED` | `PROVISIONED` |
| `PROVISIONED` | `RELEASED` |
| `REJECTED` | （终态，无出边） |
| `RELEASED` | （终态，无出边） |

> **勘误注**：05 §3 原注释把 `REJECTED` 标为「IV-3: 终态」——IV-3 实际是审批超时告警（03 §III），终态约束属于 IV-2 合法路径。本文件以 03 编号为准（06 §2 已声明勘误）。

## 2. 领域事件（6 个）

载荷字段为权威版本（05 §3 `contracts/src/events.ts`）：

| 事件 | 载荷 | 触发 |
| :--- | :--- | :--- |
| `ResourceRequested` | requestId, type, spec, project, cost, timestamp | `SubmitRequest` 成功 |
| `RequestApproved` | requestId, approver, timestamp | `ApproveRequest` 成功 |
| `RequestRejected` | requestId, approver, reason, timestamp | `RejectRequest` 成功 |
| `ResourceProvisionRequested` | requestId, type, spec, project | `RequestApproved` 同事务发布 |
| `ResourceProvisioned` | instanceId, requestId, timestamp | Provisioner 成功完成 |
| `ResourceReleased` | instanceId, requestId, timestamp | `ReleaseResource` 成功 |

> spec `## Events` 表的 `ResourceRequested` 载荷列的是 `applicant`——以本文件 `cost, timestamp` 为准（代码落地形态）。

## 3. 领域服务接口

```typescript
// contracts/src/interfaces.ts — 来自 03 §领域服务接口 + 05 §4
interface Provisioner {
  provision(req: ProvisioningCommand): Promise<void>;
  release(requestId: RequestId): Promise<void>;
}

interface PricingTable {
  lookup(type: ResourceType, spec: ResourceSpec): UnitPrice | null;
}

interface QuoteCalculator {
  calc(input: { type: ResourceType; spec: ResourceSpec; days: number }): Quote;
}

interface ResourceRequestRepository {
  findById(id: RequestId): Promise<ResourceRequest | null>;
  save(request: ResourceRequest): Promise<void>;
  findByApplicant(applicant: UserId): Promise<ResourceRequest[]>;
  findPending(): Promise<ResourceRequest[]>;
}

interface ResourceInstanceRepository {
  findByRequestId(id: RequestId): Promise<ResourceInstance | null>;
  save(instance: ResourceInstance): Promise<void>;
}

interface CostRecordRepository {
  findByProject(projectId: ProjectId): Promise<CostRecord>;
  save(record: CostRecord): Promise<void>;
}
```

## 4. 值对象与聚合根

| 工件 | 类型 | 关键字段 | 来源 |
| :--- | :--- | :--- | :--- |
| `Quote` | 值对象（不持久化） | unitPrice, days, totalPrice, calculatedAt | BillingContext（D2） |
| `ResourceRequest.cost` | 不可变快照 | totalPrice 快照，创建时确定 | IV-5 + B-2 |
| `CostRecord` | 读模型 | projectId, actual, forecast, updatedAt | BillingContext（B-3） |
| `PricingTable` | 配置查找表 | (type, spec) → 单价/天，Ops 维护 | BillingContext |

## 5. Mock → Real 切换点

| 实现 | 形态 | 切换时机 |
| :--- | :--- | :--- |
| `MockProvisioner` | `setInterval` 5s 模拟异步完成 | P5（本阶段） |
| `AliyunProvisioner` / `AWSProvisioner` | 调用真实云 SDK | P7 部署交付 |

三者实现同一 `Provisioner` 接口；契约测试 `provisioner.contract.ts` 覆盖 5 状态转换（D1，详见 ADR-001）。MVP 的 localStorage + setInterval 方案见 ADR-004。
