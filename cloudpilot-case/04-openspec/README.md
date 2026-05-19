# 04 · OpenSpec 输出

> **阶段**：AI-Native DevOps P4 OpenSpec 规范定义
> **上游输入**：[`../03-ddd-modeling.md`](../03-ddd-modeling.md) §V `@ddd-openspec-bridge`
> **下游消费**：P5 代码生成（spec → code）、P6 评审、P7 部署

## 文件结构

```
04-openspec/
├── proposal.md              # Why / What Changes / Impact
├── design.md                # 架构 + 集成契约 + 关键决策
├── tasks.md                 # 实现任务拆解（7 阶段）
└── specs/
    ├── resource-request/spec.md       # IV-1 ~ IV-6（核心状态机）
    ├── resource-management/spec.md    # IV-7, IV-8 + Provisioner 契约
    └── billing/spec.md                # 报价 + 项目成本
```

## 来源映射（从 DDD 工件到 OpenSpec 工件）

| DDD 工件                              | OpenSpec 文件                                 |
| :------------------------------------ | :-------------------------------------------- |
| `@ddd-scope` problem + goals          | [`proposal.md`](./proposal.md) §Why / What    |
| `@ddd-subdomains` + `@ddd-contexts`   | [`proposal.md`](./proposal.md) §What Changes  |
| `@ddd-context-map`                    | [`design.md`](./design.md) §集成与契约        |
| `@ddd-aggregates` IV-1 ~ IV-6         | [`specs/resource-request/spec.md`](./specs/resource-request/spec.md) |
| `@ddd-aggregates` IV-7 ~ IV-8         | [`specs/resource-management/spec.md`](./specs/resource-management/spec.md) |
| `BillingContext` + `QuoteCalculator`  | [`specs/billing/spec.md`](./specs/billing/spec.md) |
| `@ddd-domain-interactions` 仓库 / 服务接口 | [`tasks.md`](./tasks.md)                  |

## 阅读建议

- **产品 / 业务**：先读 [`proposal.md`](./proposal.md)
- **架构 / 资深工程师**：[`design.md`](./design.md) + 3 份 spec
- **开发 / AI 代码生成 Agent**：[`tasks.md`](./tasks.md) + 3 份 spec（spec 是单一事实源）
