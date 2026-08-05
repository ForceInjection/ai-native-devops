# SDD 实践：AI Native DevOps 基于 《SDD 实战》一书取长补短

> 「规范是第一手工件」——这是 CloudPilot 案例与《SDD 实战》共享的起点。但在这份共识之后，两条路分开了：我们在 AI Native DevOps 框架下自研的范式，用 **DDD + OpenSpec 双层结构**与治理制度保证质量；黄佳的 sdd-in-action 用**习惯与生命周期机制**保证演进。
>
> 本文站在我们的范式上，审视对方设计中值得吸收的部分——演进机制、契约与决策工件——并记录一次真实的 change 生命周期（`openspec init` → `openspec archive`）里，每个关键决策的权衡。

---

## 1. 我们的范式：治理为内核的设计与一个盲区

### 1.1 设计取向：制度保证质量

AI Native DevOps 是规范驱动开发的一种自研范式，立足点「AI 增强而非替代」决定了它的设计取向：**质量靠制度，不靠运气**。制度落在两条线上——治理制度（8 个阶段、人工确认点、RACI）与建模制度（DDD 门禁），后者构成技术主线：**DDD + OpenSpec 的双层结构**——P3 用九段 DDD 流水线做领域建模，聚合不变量收敛为 IV-N 编号；P4 通过 `@ddd-openspec-bridge` 把领域工件桥接为 OpenSpec 规范。领域语义先在建模层收敛、过门禁，再进入可校验的规范层——这是本范式区别于其他 SDD 实现的核心：**在 OpenSpec 之上叠加了领域建模这一层**。

8 个阶段（P1 愿景→PRD 到 P8 变更演进）每个都显式标注 AI 输入、AI 输出、建议工件与人工确认点，RACI 责任矩阵把最终决策、风险承担与跨团队仲裁指派给人工 Owner；CloudPilot 案例是这个范式最完整的落地：P1-pre 访谈、P2 可交互 Mock、P3 九段 DDD 流水线（IV-N 不变量门禁）、P4 OpenSpec 规范集、P6 代码桥接与 P7 实现工作流（Stage A→E），一路走到可运行代码。

规范侧的家底是本文实践的对象：`04-openspec/` 四类工件（proposal/design/tasks/3 份 spec）由 `@ddd-openspec-bridge` 从 DDD 模型直接生成，resource-request / resource-management 的每个 Requirement 挂 IV-N 不变量、挂 Scenario；`config.yaml` 用 24 条规则约束 AI 的生成行为（SHALL/MUST 语言、frontmatter 五字段、命名约定），下游实现文档可以逐条映射回规范。

### 1.2 盲区：工件类型不完整，演进机制未工具化

盘点完家底，自然的问题是：这个范式还差什么？说「缺」不准确——更准确的说法是**设计取舍的盲区**。我们的范式把治理与建模设计得很重（人工确认点、IV-N 门禁、config.yaml 的 24 条规则），但规范驱动在完整性与演进性两个维度上都有未覆盖的区域：

- **工件类型不完整**：规范只有 proposal/design/tasks/specs 四类，没有契约、决策记录、编号溯源这三类工件——
  1. **契约只存在于代码示例**。`05-p5-code-bridge.md` §3 的 `@cloudpilot/contracts`（状态枚举、VALID_TRANSITIONS、6 个领域事件、Provisioner 接口）是规范最重要的边界，却没有独立工件——design 和 tasks 无法共同引用它，接口字段以哪份文档为准没有裁决。
  2. **关键决策无留痕**。`design.md` 的 D1-D4 是架构选型结论，但「为什么选、否决了什么、影响哪些文件」没有记录。
  3. **billing 规范无编号**。billing 的 4 个 Requirement 没有不变量编号，验收门禁无法精确引用它。
- **演进机制未工具化**：「变更与演进」被设计成 P8 阶段——一个文档描述的阶段，没有配套的机制—— 4. **生命周期从未闭环**。`04-openspec/` 是手工维护的「OpenSpec 风格」目录：没有 `openspec/changes/`，没有 `project.md`，Stage E 的 `openspec archive` 在案例重放中被跳过——规范只有「出生」（AI 生成）和「展示」，没有「变更、验证、归档」的演进路径。

两个维度不是并列的：**机制缺失恰恰是工件类型长期未被发现的原因**——没有变更流程，规范不会定期被重新审视，缺了什么就永远不会有人注意到。

这个盲区的代价有一个现成的证据：校对时在 billing spec 里发现「UI 禁用提交按钮（FR-08）」——FR-08 实为项目成本中心，与禁用提交无关；`04-openspec/tasks.md` 里「实时报价调用 QuoteCalculator（FR-03）」也是错的（实时报价是 FR-02）。这类引用错误在静态文档里不会自己暴露——没有校验关卡，错误只会在被引用时被发现，或者永远不被发现。

---

## 2. 另一种 SDD 实现：sdd-in-action 的两个设计取向

### 2.1 共同的根：三次复兴

规范先行不是新事，黄佳把它放进三次复兴的谱系：1988 年 Bertrand Meyer 的契约式设计（工具消失，方法论留进 Java 的 assert、Python 的 typing、Rust 的所有权系统）；2015 年 OpenAPI 推动 API-First（先写 spec 再写代码成为微服务纪律）；当下第三次——OpenSpec、Spec-Kit、BMAD、Superpowers 几乎同时出现，回应同一事实：**AI 写代码的成本越来越低，真正稀缺的是清晰、可执行、可验证的意图**。

两种范式都是第三次复兴的产物——都选择了 OpenSpec 不是巧合，而是共享这份共识：规范是 AI 进入工程闭环的桥接点。分歧在桥接点之后：我们叠加 DDD 建模层，黄佳范式从 proposal 直接进 design，不经过领域建模。

### 2.2 取向一：质量靠习惯（95/5 原则与工具金字塔）

黄佳范式最反直觉的设计取向是价值分布：**95% 的价值来自「动手前先坐下来把需求想清楚，写成结构化文档交给 AI」，剩下 5% 才是工具的事**。工具不重要到什么程度？「OpenSpec / Spec-Kit / Superpowers / BMAD 哪个都行，真的，哪个都行」。

价值分布划定了 95/5，剩下 5% 的工具怎么花，他的答案是按层级选——工具四层金字塔（L1 原子技能 → L2 上下文持久化 → L3 规范框架 → L4 完整方法论）：想给 AI 加「先追问再动手」的约束，L1 的六行 Markdown 就够；想要全流程可审计，才值得上 L4。**工具的层级必须匹配问题的层级**，错配就是在 5% 的事情上花 95% 的时间。这与本框架 Architecture 层的 Agent/Skill/Tool 成本金字塔（「不是处处需要 LLM」，能用 Tool 就不用 Agent）是同一条纪律的两面，只是视角从「嵌入产品系统」换成了「选择开发工具」。

两条判断在本案例上各有用途：95/5 解释了「规范内容为什么值得花 95% 的精力」（这是本文实践的动机）；金字塔回答了「P4 为什么锁定 OpenSpec」——OpenSpec 是 L3「规范框架」（轻量迭代、工具无关），匹配 CloudPilot 单仓、单团队、规范由 AI 生成的场景；BMAD 的 L4（多角色编排）和 Superpowers 的 L4（全流程强制）对这个体量是过度配置。

### 2.3 取向二：演进是日常机制，不是阶段

两种范式都承认「规范要演进」，但设计的位置不同：我们的范式把它设计成 P8 阶段；黄佳范式把演进设计成**日常机制**——一次变更 = 一个 `openspec/changes/<name>/` 目录（proposal=为什么 / specs=什么 / design=怎么做 / tasks=分几步），从 `new` 到 `archive` 走完就是一个 PR 周期；`project.md` 是每次执行必读的项目上下文；归档后 change 移入 `changes/archive/`，半年后可回读原始动机。行动营 Week 2 的结论是十条命令里高频只有四条（`new/ff/apply/verify`），以及一条被实践反复验证的经验：**每个 PR 对应一个 change，归档即留痕**。

这是对方设计中最值得吸收的部分——它直接补上 §1.2 的机制盲区：我们的治理制度保证了「生成时不跑偏」，但没有机制保证「演进时不腐烂」。

---

## 3. 吸收设计：站在我们的范式上

### 3.1 吸收不是全盘接受

取长补短的前提是主位清晰：我们吸收对方的设计取向（演进机制、契约/决策工件），同时保留自己范式的内核（治理制度、DDD 门禁、编号体系）。「保留」落在三条约束上——一条来自处境、一条来自制度、一条来自现状：

- **评估基准只读（处境）**。`04-openspec/` 和 03/05/06 文档是 Golden Set——`对比` 命令的比对基准，「方法论正确执行后的参考产出物」。吸收只能在副本上进行。
- **不编造编号（制度）**。IV-N 的唯一来源是 DDD 模型——模型只有 IV-1~IV-8 八个不变量，BillingContext 没有不变量表，所以 billing 不能凭空挂 IV-9。这是 DDD 门禁的制度延伸。
- **两套语法分清（现状）**。04-openspec 的「中文引用块 frontmatter + `## ADDED Requirements` 头」是 AI 提示约定（config.yaml 注入）；CLI 的主基线要求 `## Requirements` + `## Purpose`（Priority/Rationale 是约定行，解析不校验）。这不是制度，是客观现状，但决定了迁移必须显式转换。

### 3.2 工件映射

| 对方范式的工件             | 我们原本的对应物                                        | 本工作区落点                  | CloudPilot 内容                    |
| :------------------------- | :------------------------------------------------------ | :---------------------------- | :--------------------------------- |
| `proposal.md`（需求规范）  | `04-openspec/proposal.md`（已有，格式不同）             | `changes/<name>/proposal.md`  | Why / What / Out of scope          |
| `design.md`（架构设计）    | `04-openspec/design.md`（已有，D1-D4 是章节非独立工件） | `changes/<name>/design.md`    | 布局论证、B-N 派生策略、merge 约束 |
| `tasks.md`（任务拆解）     | `04-openspec/tasks.md`（已有）                          | `changes/<name>/tasks.md`     | 5 个 Phase，CLI 按勾选检查完成度   |
| `contracts/data-models.md` | **不存在**——契约散落在 05 §3 代码示例                   | `openspec/contracts/`（常驻） | 状态矩阵、6 事件、接口、值对象     |
| `adrs/001-005`（七段式）   | **不存在**——D1-D4 是 design.md 的章节                   | `openspec/adrs/`（常驻）      | D1-D4 提升 + 编号溯源元决策        |
| 主基线 `specs/`            | `04-openspec/specs/`（手工维护，无 CLI 校验）           | `openspec/specs/`（CLI 校验） | 迁移转换后的 3 份 capability       |

表格读出的吸收结构：前三行是「同工件、换格式」，后三行是「**从无到有**」——contracts、adrs、CLI 校验的主基线，是我们原本完全没有的工件类型，这正是 §1.2 说的工件类型盲区。

### 3.3 三个关键决策

**决策一：contracts/ 与 adrs/ 放哪里——常驻顶层，还是随 change 走？**

三个候选位置各有代价：

- **放 change 目录内**：契约跟着这次变更归档。问题是契约和决策是主基线的长期资产，不是某次变更的产物——归档后半年，`openspec/contracts/` 不存在，契约反而比现在更不可达。
- **放工作区外**（与 04-openspec 并列）：脱离 CLI 上下文，`project.md` 和 tasks 引用它需要跨目录跳，又回到「工件散落、无统一入口」的老问题。
- **常驻 `openspec/` 顶层**：CLI 只消费 `openspec/specs` 与 `openspec/changes`，顶层未知目录不影响 validate/doctor（实测确认），成本为零；契约与决策随工作区版本管理，与主基线同生命周期。

结论：常驻顶层。这个决策的本质是**把 SDD book-code 的「contracts/ 被 design 与 tasks 共同引用」从文档惯例升级为工作区布局**——代价是 CLI 不校验这两个目录的内容，质量靠人 review 保证，这正是 §5 要讨论的边界。

**决策二：billing 编号怎么办——回溯 DDD，派生编号，还是维持无编号？**

这是吸收过程中取舍最重的一个决策。三个方向：

- **回溯 ddd-modeler 给 BillingContext 补不变量表，挂 IV-9~IV-12**。诱惑最大：编号体系统一，不用发明新规则。代价也最大：`03-ddd-modeling.md` 是评估基准，修改它等于让「被评估的产物」反过来改「评估标准」——既当运动员又当裁判。且 config.yaml 的规则明写「不新增 DDD 模型未提及的概念」，回溯违背规则初衷。
- **维持无编号**。成本为零，但 billing 继续游离在门禁之外，§1.2 的表现 3 等于没修。
- **从 design 关键决策 D1-D4 派生可测约束，编号 B-1~B-4**。代价是引入双编号体系（IV-N 与 B-N），需要文档解释两者的来源；收益是每个 Requirement 都可被精确引用，且「派生」不新增 DDD 模型未提及的概念——billing 的四个 Requirement（实时报价、报价快照、成本汇总、ACL 防腐）恰好都能对应到 design 决策：B-1（D2/D4 + FR-02/NFR-01 的 500ms 上限）、B-2（D2 + 联动 IV-5）、B-3（D3 事件驱动）、B-4（D1 + context-map 模式）。

结论：B-N 派生，且编号溯源规则写成 ADR-005——「为什么否决回溯」本身成为留痕的一部分，双编号体系的可解释性由这份 ADR 兜底。

**决策三：编号标注放标题还是正文——merge 按名匹配的约束**

最初把 delta 标题写成「实时报价（B-1）」，`openspec validate` 通过了。到 `openspec archive` 时被拦下：

```text
billing MODIFIED failed for header "### Requirement: 实时报价（B-1）" - not found
Aborted. No files were changed.
```

merge 按 Requirement 名与主基线匹配替换——标题不同名，会变成「新增一条 + 残留旧条目」，而不是「替换」。这是工具约束，不是可选项；它逼出的设计选择是：**编号放正文与 Rationale，标题保持与主基线同名**。

这个失败包含两层信息：validate 通过而 archive 失败，说明两个关卡校验的不是同一层——validate 校验语法结构，archive 校验与主基线的匹配关系。如果只跑 validate 就归档（Stage E 在案例重放中正是这样被跳过的），错误不会暴露；archive 的「先 validate 再 merge」顺序本身就是一次验收关卡。

### 3.4 反向：我们的范式能给对方什么

取长补短是双向的。对方范式用习惯保证质量（95/5），我们用制度——人工确认点、RACI、IV-N 门禁、config.yaml 规则注入。当团队规模变大、或规范要跨团队共享时，习惯的约束力会衰减（黄佳书第 9 章「个人 → 团队 → 组织」的落地路径，正需要治理机制来接住）；同样，**DDD + OpenSpec 双层结构**是我们相对对方的增量——对方范式从 proposal 直接进 design，没有领域收敛与不变量门禁，IV-N 编号体系（对方的 spec 用 Scenario 驱动，没有可跨工件引用的编号）也无从谈起。吸收对方设计的同时保留这些内核，才是取长补短——本文的实践只吸收了对方的一半（演进机制与契约/决策工件），DDD 建模层与治理制度全部保留。

---

## 4. 实践实录：从 onboard 到 archive

### 4.1 工作区搭建

```bash
cd cloudpilot-case
openspec init . --tools none
```

产出 `openspec/{specs,changes,changes/archive}/` + `openspec/config.yaml`。随后手工写 `openspec/project.md`——从行动营移植的工件：项目定义、技术栈、关键数据契约、项目约定（IV-N 唯一来源、SHALL/MUST、B-N 派生规则）、**已知 gap 清单**、术语表。gap 清单后来直接成了 change 的选题——已知 gap 是 change 的 backlog。

主基线迁移有三种做法，选择了「复制 + 转换」而非「原地改造」或「重新生成」：原地改造会破坏 Golden Set 的对比基准；重新生成会引入新的 AI 输出、与基准失去可比性。复制 + 转换保留全部语义，只改约束三（两套语法）要求的格式差异：

| 04-openspec 现状                  | 工作区目标                            | 为什么改                                                                             |
| :-------------------------------- | :------------------------------------ | :----------------------------------------------------------------------------------- |
| `# Spec · billing`                | `# billing Specification`             | 与 CLI 解析器期望的标题形态一致                                                      |
| `## ADDED Requirements`           | `## Requirements`                     | 实测：delta 头在主基线会被 validate 判为 issue                                       |
| 无 `## Purpose`                   | 补，≥50 字符                          | CLI 硬约束（MIN_PURPOSE_LENGTH）                                                     |
| Requirement 无 Priority/Rationale | 补 `**Priority**:` / `**Rationale**:` | 约定行（解析不校验），但人读需要——`openspec show` 与 review 都依赖它定位优先级与来源 |

迁移后校验通过：`openspec validate --specs` → 3 passed, 0 failed。

### 4.2 change 生命周期

```bash
openspec new change enrich-specs-contracts-adrs --description "契约与决策显式化…"
```

四工件手工填充后，生命周期逐关卡推进（每步输出均为真实结果）：

```text
$ openspec validate enrich-specs-contracts-adrs --json
→ valid: true, issues: []                        # 关卡 1：语法

$ openspec change show enrich-specs-contracts-adrs --json --deltas-only
→ deltaCount: 4（billing MODIFIED ×4 正确解析）  # 关卡 2：解析

$ openspec status --change enrich-specs-contracts-adrs
→ Progress: 4/4 artifacts complete               # 关卡 3：完成度

$ openspec archive enrich-specs-contracts-adrs --yes
→ Applying changes to openspec/specs/billing/spec.md: ~ 4 modified
→ Totals: + 0, ~ 4, - 0, → 0                     # 关卡 4：归档
→ Change archived as '2026-08-05-enrich-specs-contracts-adrs'
```

「验证即关卡」不因变更大小打折：本 change 不写一行代码，但四个关卡一个不少。归档后的终态：`openspec validate --all` 3 passed、`openspec doctor` root ok、`openspec list` 无活动 change、`openspec show billing` 可见 B-1~B-4。

### 4.3 契约与决策落地

`openspec/contracts/data-models.md` 把 05 §3 的契约设计沉淀为常驻工件，并裁决了一处此前悬而未决的冲突：spec Events 表的 `ResourceRequested` 载荷列 `applicant`，而 05 契约是 `cost, timestamp`——本文件定为权威（代码落地形态），Events 表降级为「面向场景的简化视图」。另勘误一处编号：05 §3 把 REJECTED 终态注释为「IV-3」，实际属于 IV-2 合法路径约束（IV-3 是超时告警）——以 03 模型为准。

五份 ADR 把 D1-D4 从设计文档的章节提升为独立工件：001 Provisioner 接口对齐（D1）、002 Quote 快照不持久化（D2）、003 领域事件最终一致（D3）、004 localStorage+setInterval MVP（D4）、005 IV-N/B-N 编号溯源规则（元决策）。每份 ADR 的理由逐条引用 spec 条款——「为什么」与「是什么」在同一仓库里互相可查。

---

## 5. 权衡与教训

这次实践的整体结论可以压缩成一句话：**吸收是可行的，代价是可枚举的，边界是结构性的**。5.1 的每一条（生命周期、留痕、关卡、backlog、让工具教）都是对方机制里直接拿走就能用的部分；5.2 的每一条摩擦（两套语法、Golden Set 矛盾、命名撞车、TBD 骨架）都不是执行失误，而是两种范式相遇时的结构性摩擦——它们不会被「更仔细地做」消除，只能被「更清楚地界定」管理；5.3 的边界（工具管生命周期、人管内容）解释了摩擦为何必然存在：我们的治理制度与对方的习惯机制，在「内容判断」这一点上殊途同归——都必须由人完成，工具只是把判断的时机和形式变得更可控。取长补短之后两种范式的关系因此很明确：**机制互相吸收，内核各自保留**。

### 5.1 值得吸收的

1. **工件先问生命周期，再选位置**：contracts/ + adrs/ 常驻 `openspec/` 顶层，因为它们是主基线的长期资产而非某次变更的产物——判断一个工件该随 change 走还是常驻，先问「归档后它还需要吗」。
2. **否决路径也要留痕**：ADR-005 记录的不仅是 B-N 派生规则，更是「为什么否决回溯 ddd-modeler」——被否决的方案往往比被采纳的更值得写下来，半年后回看才有上下文。
3. **验证关卡不因变更大小打折**：本 change 不写一行代码，validate/archive 的关卡一个不少——单文档变更与代码变更走同样的生命周期，这个不变量是机制可信的基础。
4. **把 backlog 写进仓库**：project.md 的「已知 gap」清单让新成员一眼看到「下一步做什么」——backlog 存在 repo 里，不依赖看板或对话记忆。
5. **让工具教，别让文档教**：MODIFIED 同名约束是 `Aborted. No files were changed.` 教出来的（§3.3 决策三）——CLI 的失败输出是最好的文档，比任何「注意事项」章节都精确。

### 5.2 不适配的

1. **两套语法并存是常态**：AI 提示约定（中文 frontmatter、ADDED 头）与 CLI 规范（Requirements、Purpose）服务同一份规范集的两个读者——AI 生成与工具校验。分清并显式转换是吸收的前提。
2. **Golden Set 与生命周期天然矛盾**：评估基准不能进 change，只能做副本进工作区，于是「主基线漂移」必然发生。解法是 project.md 里写清二者关系，把 diff 对照当特性而非缺陷。
3. **config.yaml 命名撞车**：案例级 `cloudpilot-case/config.yaml`（案例自建的 AI 提示注入，24 条规则）与 CLI 的 `openspec/config.yaml`（工作区配置，同样支持 context/rules 注入字段）同名不同源——两套提示注入并存。吸收方向是让 `openspec/config.yaml` 吸收案例级 context/rules，只留一份真相源。
4. **归档骨架是 TBD**：`openspec new change` 生成的是占位符，`## Purpose` 等硬约束要人工补——CLI 帮你建目录，不帮你写内容。

### 5.3 边界在哪

这几条不适配划出的边界很具体：**工具管生命周期，人管内容**。§1.2 的 FR-08 引用错误是一个实例——CLI 不会发现「禁用提交按钮」引用了一个与它无关的 FR 编号，这种错误只有 review 能拦住；同理，B-N 派生的「为什么不挂 IV-9」是 DDD 门禁下的内容判断，MODIFIED 同名约束是工具规则，但「编号放正文还是标题」是设计决策。工具把「变更-验证-归档」做成关卡，但关卡之间全是需要人判断的内容工作——这正是 95/5 原则在实践中的形态，也是 DDD 建模层存在的理由：它把一部分「内容判断」前移到建模期，用 IV-N 门禁先拦一道。

### 5.4 后续

- **P6 契约包同步**：`openspec/contracts/data-models.md` 与 05 §3 的 TypeScript 实现（`@cloudpilot/contracts`）保持同步——契约工件是设计层，代码是衍生物。
- **IV-9 归并**：若 FR-11（到期告警）加入 DDD 模型产生 IV-9，按 ADR-005 的归并规则处理 B-N 与 IV-N 的重叠。
- **成本治理 spec**：行动营 Week 2 的 cost-budget 三段式（预算分层/按比例告警/自动降级）尚未移植。
- **`/cloudpilot-demo` 接入**：可扩展「对比工作区」检查，把 `openspec validate` 变成演示的一环，让 CLI 校验参与评估。
- **Golden Set 的 FR 引用修正**：billing spec「FR-08」与 tasks.md「FR-03」两处引用错误留待后续 change 修正（基准只读，不能原地改）。

---

## 参考资料

[1] `sdd-in-action` — GitHub：[github.com/huangjia2019/sdd-in-action](https://github.com/huangjia2019/sdd-in-action)（《SDD 实战》配套仓库：book-code specs 模板、行动营 week1-4）

[2] 《SDD 实战》图书配套页：[kage-ai.com/sdd](https://kage-ai.com/sdd/)（三次复兴、工具对比）

[3] SDD 行动营 Week 1 进阶问答：《SDD 的 95/5 原则》《SDD 工具生态四层金字塔》；Week 2《OpenSpec 命令优先级》

[4] [`./ai-native-devops.md`](./ai-native-devops.md)（本框架主文章，8 阶段定义与治理机制）

[5] [`../ai-native-architecture/ai-native-architecture.md`](../ai-native-architecture/ai-native-architecture.md)（Agent/Skill/Tool 成本金字塔）

[6] `OpenSpec-practise` — GitHub：[github.com/ForceInjection/OpenSpec-practise](https://github.com/ForceInjection/OpenSpec-practise)（多语言实现案例）
