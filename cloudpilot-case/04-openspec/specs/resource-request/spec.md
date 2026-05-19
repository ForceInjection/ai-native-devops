# Spec · resource-request

> **Capability**：资源申请审批（Core 子域）
> **上游来源**：[`../../../03-ddd-modeling.md`](../../../03-ddd-modeling.md) §III 战术 `@ddd-aggregates` 聚合 1（IV-1 ~ IV-6）
> **责任聚合**：`ResourceRequest`

---

## ADDED Requirements

### Requirement: 状态枚举受限（IV-1）

`ResourceRequest.status` SHALL 仅取 `PENDING | APPROVED | REJECTED | PROVISIONED | RELEASED` 五个值之一。

#### Scenario: 创建后初始状态为 PENDING

- **WHEN** 申请人调用 `SubmitRequest(type, spec, days, project)`
- **THEN** 系统创建 `ResourceRequest`，`status = PENDING`
- **AND** 发布 `ResourceRequested` 事件

#### Scenario: 拒绝非法状态值

- **GIVEN** 任意外部输入尝试将 `status` 设为字符串 `"ARCHIVED"`
- **WHEN** 通过应用服务命令进入聚合
- **THEN** 编译期类型错误 / 运行期断言失败

---

### Requirement: 状态转换合法路径（IV-2）

状态转换 SHALL 仅遵循：`PENDING → APPROVED`、`PENDING → REJECTED`、`APPROVED → PROVISIONED`、`PROVISIONED → RELEASED`。

#### Scenario: 审批后进入 APPROVED

- **GIVEN** `ResourceRequest.status = PENDING`
- **WHEN** 审批人调用 `ApproveRequest(requestId, approver)`
- **THEN** `status = APPROVED`
- **AND** 发布 `RequestApproved` 与 `ResourceProvisionRequested` 事件

#### Scenario: 拒绝后进入 REJECTED 终态

- **GIVEN** `ResourceRequest.status = PENDING`
- **WHEN** 审批人调用 `RejectRequest(requestId, approver, reason)`
- **THEN** `status = REJECTED`
- **AND** 发布 `RequestRejected` 事件
- **AND** 后续任何命令对该聚合无效

#### Scenario: 拒绝从 REJECTED 反向转换

- **GIVEN** `ResourceRequest.status = REJECTED`
- **WHEN** 调用 `ApproveRequest`
- **THEN** 抛出 `IllegalStateTransition` 异常

#### Scenario: 监听 ResourceProvisioned 推进到 PROVISIONED

- **GIVEN** `ResourceRequest.status = APPROVED`
- **WHEN** 收到 `ResourceProvisioned(requestId)` 事件
- **THEN** `status = PROVISIONED`

---

### Requirement: 审批超时告警（IV-3）

`ResourceRequest` 在 `APPROVED` 状态停留 SHALL NOT 超过 30 分钟未变 `PROVISIONED`。

#### Scenario: 30 分钟未配置触发告警

- **GIVEN** `ResourceRequest.status = APPROVED`，`approvedAt + 30min < now`
- **WHEN** `ApprovalTimeoutMonitor` 巡检
- **THEN** 推送告警到审批人 IM
- **AND** 不改变聚合状态（仅观测）

---

### Requirement: 提交幂等（IV-4）

同一 `requestId` 重复 `SubmitRequest` SHALL 仅产生一条记录。

#### Scenario: 重复提交返回已存在

- **GIVEN** 仓库中已有 `requestId = R-001` 的记录
- **WHEN** 再次调用 `SubmitRequest` with `requestId = R-001`
- **THEN** 返回已存在记录，不发新事件

---

### Requirement: 成本不可变（IV-5）

`ResourceRequest.cost` SHALL 在创建时确定，后续生命周期内不可变更。

#### Scenario: 拒绝外部修改 cost

- **GIVEN** `ResourceRequest` 已创建，`cost = 720`
- **WHEN** 任意命令尝试修改 `cost`
- **THEN** 编译期不存在 `setCost` 方法 / 运行期断言失败

---

### Requirement: 释放权限受限（IV-6）

`ReleaseResource` SHALL 仅由原申请人触发。

#### Scenario: 申请人释放成功

- **GIVEN** `ResourceRequest.status = PROVISIONED`，`applicant = U-A`
- **WHEN** `U-A` 调用 `ReleaseResource(requestId)`
- **THEN** `status = RELEASED`
- **AND** 发布 `ResourceReleased` 事件

#### Scenario: 非申请人释放被拒

- **GIVEN** `applicant = U-A`
- **WHEN** `U-B` 调用 `ReleaseResource(requestId)`
- **THEN** 抛出 `PermissionDenied`

---

## Events

| 事件                         | 触发条件                     | 载荷                                      |
| :--------------------------- | :--------------------------- | :---------------------------------------- |
| `ResourceRequested`          | `SubmitRequest` 成功         | requestId, applicant, type, spec, project |
| `RequestApproved`            | `ApproveRequest` 成功        | requestId, approver, approvedAt           |
| `RequestRejected`            | `RejectRequest` 成功         | requestId, approver, reason               |
| `ResourceProvisionRequested` | `RequestApproved` 同事务发布 | requestId, type, spec                     |
| `ResourceReleased`           | `ReleaseResource` 成功       | requestId, releasedAt                     |

## Repository

```typescript
interface ResourceRequestRepository {
  findById(id: RequestId): Promise<ResourceRequest | null>;
  save(request: ResourceRequest): Promise<void>;
  findByApplicant(applicant: UserId): Promise<ResourceRequest[]>;
  findPending(): Promise<ResourceRequest[]>;
}
```
