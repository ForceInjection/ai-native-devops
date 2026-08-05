# 04 · OpenSpec 输出

> **阶段**：AI-Native DevOps P4 OpenSpec 规范定义
> **上游输入**：[`../03-ddd-modeling.md`](../03-ddd-modeling.md) §V `@ddd-openspec-bridge`
> **下游消费**：P5 代码生成（spec → code）、P6 评审、P7 部署

> **⚠️ 与 [`../openspec/`](../openspec/) 工作区的关系**：本目录是 **P4 阶段工件（评估基准 Golden Set）**——AI 生成的规范参考产出物，`对比` 命令的比对基准，**只读不改**。[`openspec/`](../openspec/) 是它的 **CLI 实况工作区**（`openspec init` 生成的运行时状态：可校验的主基线 + changes/archive 生命周期，目录名由 CLI 固定）。工作区主基线迁移自本目录（复制+转换，`## ADDED Requirements` → `## Requirements`、补 Purpose/Priority/Rationale），后续演进（如 billing 的 B-1~B-4 派生编号、contracts/+adrs/）只发生在 `openspec/`。两者可 diff 对照、互不覆盖。

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
