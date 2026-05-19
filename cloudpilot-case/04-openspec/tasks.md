# Tasks · CloudPilot MVP

> **阶段**：AI-Native DevOps P4 OpenSpec → P5 实现的桥
> **上游输入**：[`./design.md`](./design.md) + [`./specs/`](./specs/) 三份 capability spec
> **下游消费**：P5 代码生成 Agent / 工程师拆解
> **责任人**：Tech Lead 审核任务粒度；AI 按"接口 → 实现 → 测试"模板生成
> **AI 草稿置信度**：中（任务粒度依团队约定可调）

---

## 阶段 1：基础骨架（W1）

- [ ] 1.1 初始化仓库结构：`packages/{resource-request,resource-management,billing,shared}`
- [ ] 1.2 共享类型定义 `shared/types.ts`：`RequestId`、`UserId`、`ResourceType`、`ResourceSpec`
- [ ] 1.3 引入事件总线抽象 `shared/event-bus.ts`（开发期内存版，生产替换 Kafka）
- [ ] 1.4 SSO 中间件接入（复用现有 `@company/sso-client`）

## 阶段 2：ResourceRequest 上下文（W1~W2，核心）

实现 [`specs/resource-request/spec.md`](./specs/resource-request/spec.md)：

- [ ] 2.1 聚合根 `ResourceRequest` + 5 状态枚举（IV-1, IV-2）
- [ ] 2.2 状态机方法 `submit / approve / reject / markProvisioned / release`，私有可见性 + 命令模式
- [ ] 2.3 值对象 `Quote`、`RejectionReason`（不可变 + 构造校验 IV-5）
- [ ] 2.4 仓库接口 + 内存实现 `InMemoryResourceRequestRepository`
- [ ] 2.5 应用服务 `ResourceRequestAppService`：权限检查（IV-6）、事务边界
- [ ] 2.6 领域事件发布：`ResourceRequested` / `RequestApproved` / `RequestRejected` / `ResourceProvisionRequested` / `ResourceReleased`
- [ ] 2.7 单元测试：每个不变量（IV-1 ~ IV-6）至少 1 个正例 + 1 个反例
- [ ] 2.8 状态机契约测试：覆盖所有合法 + 非法转换

## 阶段 3：ResourceManagement 上下文（W2，Mock）

实现 [`specs/resource-management/spec.md`](./specs/resource-management/spec.md)：

- [ ] 3.1 聚合根 `ResourceInstance`（IV-7, IV-8）
- [ ] 3.2 `Provisioner` 接口 + `MockProvisioner`（setInterval 5s 触发完成）
- [ ] 3.3 事件订阅：监听 `ResourceProvisionRequested` → 调用 `provision`
- [ ] 3.4 事件发布：`ResourceProvisioned` 回流给 ResourceRequest
- [ ] 3.5 契约文件 `contracts/provisioner.contract.ts`（D1 决策）
- [ ] 3.6 真实云 SDK 适配占位：`AwsProvisioner` / `AliyunProvisioner` 骨架（后续迭代填充）

## 阶段 4：Billing 上下文（W2，Supporting）

实现 [`specs/billing/spec.md`](./specs/billing/spec.md)：

- [ ] 4.1 `PricingTable` 配置加载（YAML → 不可变 Map）
- [ ] 4.2 `QuoteCalculator` 域服务
- [ ] 4.3 `CostRecord` 聚合：监听 `ResourceProvisioned` / `ResourceReleased`，累加项目级成本
- [ ] 4.4 ACL：UI 调用 `QuoteCalculator` 时不暴露 PricingTable 内部结构
- [ ] 4.5 项目成本查询 API：`GET /projects/:id/cost`

## 阶段 5：UI / Mock 演示（W2~W3）

- [ ] 5.1 `cloudpilot-mockup.html` 接入真实事件总线（替代 localStorage 直接写）
- [ ] 5.2 5 视图保持不变：申请单 / 审批列表 / 资源列表 / 项目成本 / 历史
- [ ] 5.3 实时报价调用 `QuoteCalculator`（FR-03）

## 阶段 6：横切关注点（W3）

- [ ] 6.1 审计日志中间件：状态机每次转换落审计表
- [ ] 6.2 `ApprovalTimeoutMonitor` 域服务（IV-3 告警）
- [ ] 6.3 可观测性：3 个指标 + Prometheus 导出
- [ ] 6.4 集成测试：3 个端到端场景（提交→审批→配置→释放）
- [ ] 6.5 契约测试：MockProvisioner vs 真实 SDK 协议（CI 阻断）

## 阶段 7：上线准备（W3）

- [ ] 7.1 DB schema 迁移脚本（PostgreSQL）
- [ ] 7.2 Kafka topic 创建 + 死信队列配置
- [ ] 7.3 灰度方案：单团队试点 1 周 → 全量
- [ ] 7.4 回滚 Runbook：禁用 CloudPilot 入口 → 恢复 OA 工单链路

---

## 验收门禁

| 门禁         | 标准                                                 |
| :----------- | :--------------------------------------------------- |
| **代码评审** | 每个 spec scenario 有对应测试用例覆盖                |
| **CI**       | 单元测试 + 集成测试 + 契约测试全绿                   |
| **PRD 验收** | [`../02-prd.md`](../02-prd.md) §8 AC-01~AC-06 全通过 |
| **DDD 复核** | `@ddd-model-review` 报告新一轮评分 ≥ 上一版          |
