# ADR-001：Provisioner 接口对齐真实云 SDK，Mock/Real 同一接口

## 状态

已采纳

## 背景

MVP 阶段用 Mock 模拟资源配置（`setInterval` 5s 异步完成），但 P7 部署交付时要切换为真实云 SDK（阿里云 ECS/RDS/OSS/Redis/SLB）。若不提前抽象，切换会演变成对调用方的批量改动。

## 选项

| 选项 | 优点 | 缺点 |
|------|------|------|
| Mock 与真实实现各自独立，不抽象接口 | MVP 最快，无抽象成本 | P7 切换时改所有调用方；契约测试无处安放 |
| 定义 `Provisioner` 接口，Mock/Real 实现同一接口 | 调用方只依赖接口；契约测试覆盖 5 状态转换 | 早期付出抽象成本 |
| 定义接口 + 完整契约测试（`provisioner.contract.ts`） | 平滑切换 + 回归保障；真实 SDK 落地即跑测试 | 契约文件需独立维护 |

## 决策

定义 `Provisioner` 接口（`provision` / `release`），Mock 与真实 SDK（Aliyun / AWS）实现同一接口；契约测试 `contracts/provisioner.contract.ts` 覆盖 5 状态转换，独立维护。

## 理由

1. `design.md` D1（04-openspec）：「Mock 与真实 SDK 实现同一接口；契约测试覆盖 5 状态转换」
2. `resource-management` spec「Provisioner 接口契约（D1）」：所有实现 SHALL 遵守同一接口契约，真实 SDK 通过同一契约测试
3. 05 §4「Mock → 真实 SDK 切换点」：`MockProvisioner` / `AliyunProvisioner` / `AWSProvisioner` 三者共享 `ProvisionerInterface`
4. 接口字段见 `contracts/data-models.md` §3

## 影响

- 新增：`contracts/data-models.md` 的 `Provisioner` 接口、契约测试设计
- 后续 P5 实现：`services/resource-management/src/services/` 下 `MockProvisioner.ts`（5s 延迟）与 `AliyunProvisioner.ts`（P7）
- 无主基线 spec 变更（接口契约 Requirement 已存在，D1）

---

*ADR 版本：v1.0 | 来源：design.md D1（04-openspec）*
