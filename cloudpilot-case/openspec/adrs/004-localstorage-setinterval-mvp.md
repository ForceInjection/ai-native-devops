# ADR-004：MVP 用 localStorage + setInterval 模拟，不接真实 DB / MQ

## 状态

已采纳

## 背景

CloudPilot 是演示 MVP（Mock UI + 交互演示）。存储与异步机制需要裁决：是直接上真实基础设施，还是先用浏览器能力模拟。

## 选项

| 选项 | 优点 | 缺点 |
|------|------|------|
| localStorage 存 `requests` 数组 + `setInterval` 5s 触发 `MarkProvisioned` | 演示零门槛（浏览器双击即可跑）；无部署依赖 | 不是真实持久化；单机内存态 |
| PostgreSQL + Kafka | 生产形态；真实事件流 | 演示环境搭建成本高；偏离「演示 MVP」定位 |
| 本地 JSON 文件 + cron | 比 localStorage 真实 | 需要本地服务进程；演示分发困难 |

## 决策

MVP 用 localStorage 存 `requests` 数组；`setInterval` 5s 触发 `MarkProvisioned` 模拟异步配置。正式版替换为 PostgreSQL + Kafka，**接口不变**。

## 理由

1. `design.md` D4（04-openspec）：「UI 层 localStorage 存 requests 数组；setInterval 5s 触发 MarkProvisioned，模拟异步配置」「正式版替换为 PostgreSQL + Kafka，接口不变」
2. `cloudpilot-mockup.html` 的 P2 约束：「纯静态，浏览器双击即可运行；不引入任何外部框架」
3. 接口不变性由 ADR-001 的 `Provisioner` 接口 + 契约测试保障——切换基础设施不触碰调用方
4. 事件驱动语义（ADR-003）在 Mock 下依然成立：`setInterval` 完成即发布 `ResourceProvisioned` 事件

## 影响

- 新增：`contracts/data-models.md` §5「Mock → Real 切换点」表
- 无主基线 spec 变更（行为语义由 spec 定义，存储机制是实现细节）
- 后续 P7：`MockProvisioner` 换 `AliyunProvisioner`，存储换 PostgreSQL + Kafka

---

*ADR 版本：v1.0 | 来源：design.md D4（04-openspec）*
