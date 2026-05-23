# 05 · P5 代码桥接：从 OpenSpec 到实现

> **阶段**：AI-Native DevOps P5 实现与测试
> **上游输入**：[`04-openspec/`](./04-openspec/)（3 份 spec，8 个 IV-N，26+ Scenario）
> **下游消费**：可编译运行的多服务代码库
> **参考实现**：[OpenSpec-practise](https://github.com/ForceInjection/OpenSpec-practise)（电商领域，同一套 spec 结构驱动 Node.js + Python）
> **责任人**：Tech Lead

---

## 概述

CloudPilot 案例在 P4 产出了 3 份 spec（resource-request / resource-management / billing），定义了 8 个聚合不变量和 26+ 个 Scenario。本文档说明如何将这些 spec 映射为可编译运行的代码结构——不实现具体业务逻辑，但建立 Spec 与代码的完整追溯链。

**核心方法论**：Spec 是单一事实源（Single Source of Truth），代码是对 Spec 的忠实翻译。每个 `### Requirement:` 对应一个或多个实现文件，每个 `#### Scenario:` 对应一个或多个测试用例。

参考 [OpenSpec-practise](https://github.com/ForceInjection/OpenSpec-practise) 已验证的模式：相同的 `proposal / design / tasks / specs` 结构成功驱动了 Node.js 和 Python 两套实现。

---

## 1. 项目结构

从 `04-openspec/specs/` 的三个 capability 推导出的 monorepo 结构：

```
cloudpilot/
├── contracts/                     # @cloudpilot/contracts — 共享契约包
│   ├── src/
│   │   ├── events.ts              # 6 个领域事件类型定义
│   │   ├── commands.ts            # 命令类型 (SubmitRequest, ApproveRequest, ...)
│   │   ├── status.ts              # ResourceRequestStatus 枚举 + 状态转换矩阵
│   │   └── interfaces.ts          # Repository / Service 接口
│   └── package.json
│
├── services/
│   ├── resource-request/          # Core — 对应 specs/resource-request/
│   │   ├── src/
│   │   │   ├── domain/
│   │   │   │   ├── ResourceRequest.ts          # 聚合根 (IV-1 ~ IV-6)
│   │   │   │   └── ResourceRequestStateMachine.ts
│   │   │   ├── services/
│   │   │   │   └── RequestApplicationService.ts # 命令入口
│   │   │   ├── repo/
│   │   │   │   └── ResourceRequestRepo.ts       # ResourceRequestRepository 实现
│   │   │   └── index.ts
│   │   ├── __tests__/
│   │   │   ├── ResourceRequest.test.ts          # IV-1 ~ IV-6 Scenario 测试
│   │   │   └── StateMachine.test.ts
│   │   └── package.json
│   │
│   ├── resource-management/       # Supporting — 对应 specs/resource-management/
│   │   ├── src/
│   │   │   ├── domain/
│   │   │   │   └── ResourceInstance.ts          # 聚合根 (IV-7, IV-8)
│   │   │   ├── services/
│   │   │   │   ├── MockProvisioner.ts           # Mock 实现
│   │   │   │   └── ProvisionerInterface.ts      # ← P5→P7 切换点
│   │   │   ├── repo/
│   │   │   │   └── ResourceInstanceRepo.ts
│   │   │   └── index.ts
│   │   ├── __tests__/
│   │   │   └── ResourceInstance.test.ts
│   │   └── package.json
│   │
│   ├── billing/                   # Supporting — 对应 specs/billing/
│   │   ├── src/
│   │   │   ├── domain/
│   │   │   │   ├── PricingTable.ts
│   │   │   │   └── ProjectCost.ts
│   │   │   ├── services/
│   │   │   │   ├── QuoteCalculator.ts           # < 500ms 报价
│   │   │   │   └── CostAggregator.ts            # 事件驱动成本读模型
│   │   │   ├── repo/
│   │   │   │   ├── PricingTableRepo.ts
│   │   │   │   └── ProjectCostRepo.ts
│   │   │   └── index.ts
│   │   ├── __tests__/
│   │   │   ├── QuoteCalculator.test.ts
│   │   │   └── CostAggregator.test.ts
│   │   └── package.json
│   │
│   └── audit/                     # Generic
│       ├── src/
│       │   ├── AuditEventStore.ts
│       │   └── index.ts
│       └── package.json
│
├── docker-compose.yml             # 本地开发编排
└── README.md
```

**关键设计决定**：

- `contracts/` 包是三个服务的共享语言边界——所有事件类型、命令类型、接口定义都在这里，服务之间只通过 contracts 耦合
- `MockProvisioner` 与 `ProvisionerInterface` 分离——P5 用 Mock，P7 切换真实云 SDK 时只需换实现，不改调用方
- 每个 spec 文件对应一个服务目录，每个 `### Requirement:` 对应一个或多个 `domain/` 下的文件

---

## 2. Spec → 代码映射表

### 2.1 resource-request → `services/resource-request/`

| Spec 中的 Requirement | IV-N | 对应的代码文件 | 对应的测试文件 |
| :--- | :--- | :--- | :--- |
| 状态枚举受限 | IV-1 | `domain/ResourceRequest.ts` (类型定义) | `__tests__/ResourceRequest.test.ts` |
| 状态转换合法路径 | IV-2 | `domain/ResourceRequestStateMachine.ts` | `__tests__/StateMachine.test.ts` |
| REJECTED 终态不可再激活 | IV-3 | `domain/ResourceRequestStateMachine.ts` | `__tests__/StateMachine.test.ts` |
| 幂等提交 | IV-4 | `repo/ResourceRequestRepo.ts` | `__tests__/ResourceRequest.test.ts` |
| type+spec 引用完整性 | IV-5 | `domain/ResourceRequest.ts` (校验逻辑) | `__tests__/ResourceRequest.test.ts` |
| 审计日志 | IV-6 | `services/RequestApplicationService.ts` (事件发布) | `__tests__/ResourceRequest.test.ts` |

### 2.2 resource-management → `services/resource-management/`

| Spec 中的 Requirement | IV-N | 对应的代码文件 | 对应的测试文件 |
| :--- | :--- | :--- | :--- |
| 资源配置因果关系 | IV-7 | `domain/ResourceInstance.ts` | `__tests__/ResourceInstance.test.ts` |
| 资源释放终态约束 | IV-8 | `domain/ResourceInstance.ts` | `__tests__/ResourceInstance.test.ts` |

### 2.3 billing → `services/billing/`

| Spec 中的 Requirement | 对应的代码文件 | 对应的测试文件 |
| :--- | :--- | :--- |
| 实时报价响应 | `services/QuoteCalculator.ts` | `__tests__/QuoteCalculator.test.ts` |
| PricingTable 完整性 | `domain/PricingTable.ts` | `__tests__/QuoteCalculator.test.ts` |
| 项目成本读模型 | `services/CostAggregator.ts` | `__tests__/CostAggregator.test.ts` |
| 项目成本看板 | `services/CostAggregator.ts` | `__tests__/CostAggregator.test.ts` |

---

## 3. 契约包设计 (`@cloudpilot/contracts`)

契约包是三个服务的**共享语言边界**。来自 `03-ddd-modeling.md` 的接口定义在此落地：

```typescript
// contracts/src/status.ts
export enum ResourceRequestStatus {
  PENDING = 'PENDING',
  APPROVED = 'APPROVED',
  REJECTED = 'REJECTED',
  PROVISIONED = 'PROVISIONED',
  RELEASED = 'RELEASED',
}

// 直接编码 IV-2 合法转换路径
export const VALID_TRANSITIONS: Record<ResourceRequestStatus, ResourceRequestStatus[]> = {
  [ResourceRequestStatus.PENDING]: [ResourceRequestStatus.APPROVED, ResourceRequestStatus.REJECTED],
  [ResourceRequestStatus.APPROVED]: [ResourceRequestStatus.PROVISIONED],
  [ResourceRequestStatus.PROVISIONED]: [ResourceRequestStatus.RELEASED],
  [ResourceRequestStatus.REJECTED]: [],   // IV-3: 终态
  [ResourceRequestStatus.RELEASED]: [],   // IV-8 相关: 终态
};

// contracts/src/events.ts
// 6 个领域事件类型，直接来自 03-ddd-modeling.md §领域事件目录
export interface ResourceRequested { requestId: string; type: string; spec: string; project: string; cost: number; timestamp: string; }
export interface RequestApproved { requestId: string; approver: string; timestamp: string; }
export interface RequestRejected { requestId: string; approver: string; reason: string; timestamp: string; }
export interface ResourceProvisionRequested { requestId: string; type: string; spec: string; project: string; }
export interface ResourceProvisioned { instanceId: string; requestId: string; timestamp: string; }
export interface ResourceReleased { instanceId: string; requestId: string; timestamp: string; }
```

---

## 4. Mock → 真实 SDK 切换点（P5 → P7 演进）

这是云管平台与通用电商案例的关键差异——CloudPilot 的 Provisioner 需要从 Mock 切换为真实云 SDK。

```
P5 (本阶段)                        P7 (部署交付)
─────────────                      ─────────────
MockProvisioner                    RealCloudProvisioner
  setInterval(5s)                    → 调用云厂商 API
  内存状态                           → 真实资源生命周期
                                    AliyunProvisioner
                                    AWSProvisioner
                                    ...
```

三者共享 `ProvisionerInterface`：

```typescript
// contracts/src/interfaces.ts — 来自 03-ddd-modeling.md §领域服务接口
export interface Provisioner {
  provision(requestId: string, type: string, spec: string): Promise<ResourceInstance>;
  release(instanceId: string): Promise<void>;
}

// services/resource-management/src/services/MockProvisioner.ts
export class MockProvisioner implements Provisioner {
  async provision(requestId: string, type: string, spec: string): Promise<ResourceInstance> {
    // setInterval + 内存状态推进
  }
  async release(instanceId: string): Promise<void> {
    // 内存状态更新
  }
}

// 预留骨架 — P7 时实现
// services/resource-management/src/services/AliyunProvisioner.ts
export class AliyunProvisioner implements Provisioner {
  async provision(requestId: string, type: string, spec: string): Promise<ResourceInstance> {
    // 调用 Aliyun ECS/RDS/OSS SDK
  }
  async release(instanceId: string): Promise<void> {
    // 调用 Aliyun SDK 释放资源
  }
}
```

切换方式：`docker-compose.yml` 中注入不同实现。Mock 用于本地开发和 CI，Real 用于 staging/production。

---

## 5. Scenario → 测试用例映射示例

来自 OpenSpec-practise 的验证模式：每个 `#### Scenario:` 对应一个 `test()` 或 `it()` 块。

以 `specs/resource-request/spec.md` 中的 Scenario "同一 requestId 二次提交返回已有单" 为例：

```typescript
// services/resource-request/__tests__/ResourceRequest.test.ts
import { ResourceRequestStatus } from '@cloudpilot/contracts';

describe('ResourceRequest — IV-4 幂等提交', () => {
  // 映射自 Scenario: "同一 requestId 二次提交返回已有单"
  it('should return existing request when same requestId submitted twice', async () => {
    const repo = new ResourceRequestRepo();
    const svc = new RequestApplicationService(repo);

    const first = await svc.submit({ requestId: 'REQ-0001', type: 'ECS', spec: '4C8G', days: 30, project: 'order-service' });
    const second = await svc.submit({ requestId: 'REQ-0001', type: 'ECS', spec: '4C8G', days: 30, project: 'order-service' });

    expect(second.id).toBe(first.id);                // 返回已有单
    expect(await repo.count()).toBe(1);              // 不创建新记录
  });

  // 映射自 Scenario: "不同 requestId 创建不同记录"
  it('should create separate records for different requestIds', async () => {
    const repo = new ResourceRequestRepo();
    const svc = new RequestApplicationService(repo);

    await svc.submit({ requestId: 'REQ-0001', type: 'ECS', spec: '4C8G', days: 30, project: 'order-service' });
    await svc.submit({ requestId: 'REQ-0002', type: 'ECS', spec: '4C8G', days: 30, project: 'order-service' });

    expect(await repo.count()).toBe(2);              // 两条独立记录
  });
});
```

---

## 6. 与 OpenSpec-practise 的关系

| | CloudPilot (本文档) | OpenSpec-practise |
| :--- | :--- | :--- |
| 领域 | 云管平台 | 电商 |
| 阶段 | P5 设计（桥接文档） | P5-P7 实现（可运行代码） |
| Spec 数 | 3 份 | 6 份 |
| 实现语言 | TypeScript（设计） | Node.js + Python（双实现） |
| 关系 | 定义"应该怎么做" | 展示"已经怎么做" |

**阅读建议**：本文档描述了 CloudPilot 的 P5 代码结构和 Spec→代码映射规则。要看到这套方法论的实际运行结果，请参考 [OpenSpec-practise](https://github.com/ForceInjection/OpenSpec-practise) 仓库中的 `examples/ecommerce-mini/` 和 `examples/ecommerce-mini-python/`。
