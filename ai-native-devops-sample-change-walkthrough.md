# AI Native DevOps 模板：订单取消能力变更全链路演练

该文件定位为标准演练模板，用于把主文档中的方法、治理边界与协作规则落成一套可复用的变更样板。下面以“订单取消能力”为例，演示 AI 如何辅助需求整理、领域建模、OpenSpec 编写、实现验证、交付归档与反馈回写，同时明确必须保留的人工确认点；其中 DDD 与 OpenSpec 的方法边界基于参考项目的公开结构与工作流说明 [1, 2]。

## TOC

- [1. 场景与输入](#1-场景与输入)
- [2. 建模与规范](#2-建模与规范)
- [3. 实现与验证](#3-实现与验证)
- [4. 交付与反馈](#4-交付与反馈)
- [5. 工件清单](#5-工件清单)
- [6. 参考资料](#6-参考资料)

---

## 1. 场景与输入

标准演练模板首先固定业务边界，再给出进入建模前的最小必需输入。这样既能保证后续工件有统一上下文，也能让 AI 生成结果与人工确认点保持同一口径。

- **业务背景**：电商系统当前支持下单、支付和发货，但未提供用户主动取消订单的标准能力。
- **变更目标**：新增“支付前可取消订单”的能力，并在取消后释放库存、记录审计事件、返回统一 API 响应。
- **非目标**：本次不处理支付后的退款流程，不处理拆单、逆向物流或优惠券返还。
- **关键约束**：取消动作必须幂等；取消后状态变更必须可追踪；库存释放必须与订单状态一致。

---

### 1.1 PRD 摘要模板

以下内容可直接作为 `docs/prd/order-cancel.md` 的最小必需模板使用，用于支撑后续建模、规范与测试设计。

```markdown
## 1. 背景

用户在支付前无法主动取消订单，导致客服介入成本较高。

## 2. 目标

- 允许用户在支付前取消订单。
- 取消后自动释放库存。
- 提供统一的取消结果查询与审计记录。

## 3. 非目标

- 不处理支付后退款。
- 不处理人工审批退款链路。

## 4. 用户故事

- 作为用户，我希望在未支付时主动取消订单，避免继续占用库存。
- 作为客服，我希望看到取消原因和时间，便于后续查询。

## 5. 验收标准

- 未支付订单允许取消。
- 已支付订单调用取消接口时返回明确错误。
- 重复取消同一订单时返回幂等结果。
- 取消成功后库存恢复。

## 6. 安全与合规基线

- 数据分类：订单编号、取消原因、操作来源属于业务敏感数据；日志中不得输出用户隐私字段。
- 权限约束：仅订单拥有者和具备客服权限的角色可发起取消。
- 审计要求：必须记录取消时间、取消原因、操作来源和操作者身份。
```

### 1.2 UI/UX 输入模板

UI/UX 输入不一定需要高保真设计稿，但至少要给出关键页面状态、动作约束和异常反馈，否则后续建模很难稳定。

- **页面入口**：订单详情页显示“取消订单”按钮，仅在 `PENDING_PAYMENT` 状态可见。
- **交互提示**：点击后弹出确认框，提示“取消后将释放库存，操作不可撤销”。
- **结果反馈**：成功后展示“订单已取消”；失败时展示可解释的错误原因。
- **审计字段**：展示取消时间、取消原因、操作来源。

### 1.3 AI 辅助与人工确认点

这一阶段最适合用 AI 生成 PRD 草稿、补全验收标准并标记歧义，但进入建模前必须完成最小必需确认点。

- **AI 输入**：会议纪要、客服反馈、现有订单状态说明、库存约束。
- **AI 输出**：结构化 PRD 草稿、Given-When-Then 验收标准、非功能性需求、待确认歧义点。
- **人工确认点**：Product 确认范围和优先级，Design 确认交互约束，Architect 确认是否具备进入建模的最小必需输入。

---

## 2. 建模与规范

这一部分把需求语义收敛成领域模型，再转成 OpenSpec 变更工件。模板关注的是“从业务边界到可实现规范”的最短闭环，而不是追求一次给出完整设计。

### 2.1 子域与上下文

订单取消能力至少会涉及以下两个上下文。

| 子域 / 上下文          | 职责             | 说明                             |
| ---------------------- | ---------------- | -------------------------------- |
| `order-management`     | 订单生命周期管理 | 负责取消命令、状态迁移与审计事件 |
| `inventory-management` | 库存占用与释放   | 负责在订单取消后释放预占库存     |

### 2.2 聚合与不变量

订单取消能力的核心是订单聚合上的状态约束，因此应先从不变量出发定义边界。

- **聚合根**：`Order`
- **关键属性**：`orderId`、`status`、`reservedItems`、`cancelReason`、`cancelledAt`
- **不变量 1**：仅 `PENDING_PAYMENT` 状态允许进入 `CANCELLED`
- **不变量 2**：已取消订单不能再次发生业务状态变更
- **不变量 3**：订单取消成功后必须产生库存释放动作

### 2.3 领域命令与事件

命令与事件需要显式列出，后续才能稳定映射到 OpenSpec 能力规范、接口契约与测试用例。

| 类型    | 名称                | 说明           |
| ------- | ------------------- | -------------- |
| Command | `CancelOrder`       | 触发订单取消   |
| Event   | `OrderCancelled`    | 订单已成功取消 |
| Event   | `InventoryReleased` | 预占库存已释放 |

### 2.4 AI 辅助与人工确认点

AI 在建模阶段最适合生成候选子域、边界与事件集合，而不是替代业务专家做领域裁决。

- **AI 可辅助**：事件提取、术语对齐、候选上下文划分、领域事件清单整理。
- **人工必须确认**：核心域判断、上下文边界、聚合不变量、跨上下文协作方式。
- **输出要求**：任何被采用的模型结论都应进入 `docs/ddd/`，而不是只留在对话记录中。

---

### 2.5 OpenSpec 变更目录模板

以下目录结构可直接作为一次变更的最小必需模板使用。

```text
openspec/
└── changes/
    └── order-cancel/
        ├── proposal.md
        ├── design.md
        ├── tasks.md
        └── specs/
            ├── order-management/
            │   └── spec.md
            └── inventory-management/
                └── spec.md
```

### 2.6 `proposal.md` 模板

`proposal.md` 负责回答“为什么做”和“做什么”，以下内容可直接作为最小必需模板使用。

```markdown
## Why

当前系统不支持用户主动取消未支付订单，导致客服与库存占用成本增加。

## What Changes

- 新增未支付订单取消能力。
- 新增取消结果返回与错误语义。
- 新增取消成功后的库存释放协同。

## Capabilities

### New Capabilities

- order-management
- inventory-management
```

### 2.7 `spec.md` 模板

能力规范应把关键业务规则写成 Requirement 与 Scenario，而不是只写成任务说明。

```markdown
## ADDED Requirements

### Requirement: Cancel unpaid order

The system SHALL allow a user to cancel an order in `PENDING_PAYMENT` status.

#### Scenario: Cancel succeeds

- **Given** an order is in `PENDING_PAYMENT`
- **When** the user requests cancellation
- **Then** the order status becomes `CANCELLED`
- **And** the system records the cancellation reason and timestamp

#### Scenario: Cancel paid order is rejected

- **Given** an order is in `PAID`
- **When** the user requests cancellation
- **Then** the system rejects the request with a business error

### Requirement: Record auditable cancellation evidence

The system SHALL record cancellation reason, timestamp and operator source for every successful cancellation.

#### Scenario: Cancellation audit is stored

- **Given** an order cancellation succeeds
- **When** the system finishes state transition
- **Then** the system stores cancellation reason, timestamp and operation source
```

### 2.8 `tasks.md` 模板

`tasks.md` 应显式体现测试先行、实现、验证、安全检查与归档准备动作。

```markdown
- [ ] 为 `CancelOrder` 新增单元测试
- [ ] 为取消 API 新增集成测试
- [ ] 实现订单聚合中的取消状态迁移
- [ ] 实现库存释放协同逻辑
- [ ] 补充 OpenAPI 契约与错误码说明
- [ ] 增加取消审计字段与日志脱敏检查
- [ ] 增加权限校验与安全测试用例
- [ ] 执行 `openspec validate order-cancel`
- [ ] 完成归档前检查
```

### 2.9 AI 辅助与人工确认点

AI 适合生成 `proposal.md`、`spec.md` 与 `tasks.md` 草稿，但主流程必须保留人工兜底。

- **AI 可辅助**：从 DDD 工件生成能力草稿、补全 Scenario、拆解测试与实现任务。
- **人工必须确认**：能力边界、异常语义、错误码、权限模型、归档前条件。
- **回退规则**：若生成的 Spec 与任务不能支撑测试设计，应先修订变更工件，再进入实现。

---

## 3. 实现与验证

实现与验证负责把规范变成可运行结果，并通过测试、安全检查与契约校验建立发布信心。模板的重点不是代码细节，而是确保实现、验证和回退路径三者闭环。

### 3.1 API 契约模板

推荐把接口契约统一到 OpenAPI 描述层，这样前端、测试和文档都能消费同一份接口事实 [3]。

```yaml
paths:
  /orders/{orderId}/cancel:
    post:
      summary: Cancel an unpaid order
      operationId: cancelOrder
      parameters:
        - name: orderId
          in: path
          required: true
          schema:
            type: string
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              properties:
                reason:
                  type: string
      responses:
        "200":
          description: Order cancelled
        "409":
          description: Order cannot be cancelled in current state
        "403":
          description: Caller is not allowed to cancel this order
```

### 3.2 测试策略模板

测试至少应覆盖业务规则、幂等性、上下文协同与安全约束 4 个维度。

- **单元测试**：验证 `Order.cancel()` 的状态约束与不变量。
- **集成测试**：验证取消后库存释放与审计记录协同。
- **契约测试**：验证 API 返回码、错误体和字段结构。
- **安全测试**：验证越权取消被拒绝，日志不输出敏感字段。
- **回归测试**：验证下单、支付等既有流程没有被取消逻辑破坏。

### 3.3 验证流程模板

验证不应只依赖代码能运行，还应确保规范、任务与测试结果一致。

1. 运行 `openspec validate order-cancel`。
2. 执行单元测试、集成测试与契约测试。
3. 执行安全测试、权限检查与日志脱敏检查。
4. 检查取消接口文档与 OpenAPI 描述是否一致。
5. 复核归档条件是否满足。

### 3.4 AI 辅助与人工确认点

AI 在实现阶段最合理的角色是“基于 `tasks.md` 与契约生成实现候选”，而不是绕开规范直接写代码。

- **AI 可辅助**：生成控制器、领域服务、测试草稿、错误码映射与审查摘要。
- **人工必须确认**：公共接口语义、性能敏感路径、权限逻辑、日志与审计实现。
- **回退路径**：一旦测试、安全检查或契约校验失败，应回退到 `tasks.md` 或 `spec.md` 修订，而不是只做局部补丁。

---

## 4. 交付与反馈

交付与反馈决定一次变更能否真正形成可回溯、可审计、可持续演进的闭环。模板只保留最小发布证据、归档动作和问题回写方式，方便团队直接照此复用。

### 4.1 部署检查清单

- **部署前**：确认数据库变更、配置项和错误码更新已同步。
- **部署中**：执行标准流水线和冒烟测试。
- **部署后**：检查取消接口、库存释放链路、审计事件与权限控制是否正常。

### 4.2 归档动作模板

变更验证通过后，应执行归档并把增量规范合并回主规范目录 [2]。

```bash
# 归档模板命令。
openspec archive order-cancel
```

### 4.3 反馈回写模板

如果上线后发现“高并发下重复取消导致审计事件重复写入”，则应把该问题回写为下一轮变更输入，而不是只修代码不修规范。

- **回写位置**：新的 `proposal.md` 或缺陷类变更目录。
- **修正内容**：补充幂等性场景、并发场景和审计一致性约束。
- **治理价值**：保证问题被沉淀为知识，而不是只被局部修复。

### 4.4 最小人工签署模板

变更进入发布前，至少应保留以下人工签署与留痕。

- **Product**：确认能力范围与验收标准未偏离业务目标。
- **Architect / Tech Lead**：确认规范边界、实现策略与接口语义一致。
- **QA / Platform**：确认验证结果、发布条件、回滚路径与运行检查项满足门禁。

---

## 5. 工件清单

下面汇总与本模板对应的推荐工件，便于把演练内容直接映射到主文档建议的仓库结构中。

| 文件                                                           | 作用                           |
| -------------------------------------------------------------- | ------------------------------ |
| `docs/prd/order-cancel.md`                                     | 记录 PRD、验收标准与范围       |
| `docs/ux/order-cancel-flow.md`                                 | 记录用户旅程与界面状态         |
| `docs/ddd/order-cancel-model.md`                               | 记录上下文、聚合、不变量与事件 |
| `docs/security/order-cancel-data-classification.yaml`          | 记录数据分类与日志脱敏要求     |
| `docs/security/order-cancel-threat-model.md`                   | 记录权限模型与威胁建模结论     |
| `openspec/changes/order-cancel/proposal.md`                    | 记录变更原因与范围             |
| `openspec/changes/order-cancel/design.md`                      | 记录技术设计与数据流           |
| `openspec/changes/order-cancel/tasks.md`                       | 记录任务拆解与验证动作         |
| `openspec/changes/order-cancel/specs/order-management/spec.md` | 记录订单取消能力规范           |
| `services/api/openapi/order-cancel.yaml`                       | 记录取消接口契约               |
| `tests/contract/order-cancel.contract.test.*`                  | 记录接口契约测试               |
| `tests/security/order-cancel.authz.test.*`                     | 记录权限与安全测试             |
| `deploy/scripts/order-cancel-smoke-test.sh`                    | 记录上线后冒烟验证             |

---

## 6. 参考资料

[1] `domain-driven-design-skills` - GitHub：[github.com/ForceInjection/domain-driven-design-skills](https://github.com/ForceInjection/domain-driven-design-skills)

[2] `OpenSpec-practise` - GitHub：[github.com/ForceInjection/OpenSpec-practise](https://github.com/ForceInjection/OpenSpec-practise)

[3] `OpenAPI Specification v3.2.0` - 官方文档：[spec.openapis.org/oas/latest.html](https://spec.openapis.org/oas/latest.html)
