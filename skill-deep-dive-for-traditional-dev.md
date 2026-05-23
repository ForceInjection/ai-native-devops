# Skill 深度解读：售前内部培训手册

> **培训对象**：内部售前（Pre-sales）团队，需要在 30 分钟内向客户讲清楚 Skill 的价值并演示落地。
> **培训目标**：建立完整 What / Why / How 心智 + 掌握"常见 Agent Skill 清单"与"客户常见问题应答"。
> **前置阅读（可选）**：[`vibe-coding-intro-for-traditional-dev.md`](./vibe-coding-intro-for-traditional-dev.md) §1.4。

---

## 30 秒电梯演讲（售前可直接复用）

| 维度 | 一句话表述（可直接对客户讲） |
| :--- | :--------------------------- |
| **What** | Skill 是把团队反复粘贴的"提示词 + 步骤"封装成可复用、可审计的 SOP——本质是"提示词 + 状态机 + 工作流"的组合包。 |
| **Why** | 让 AI Agent 从"通才但不靠谱"变成"专才且可治理"——成本低于 Agent、灵活高于 Tool，是企业落地 AI 时 ROI 最快的中间层。 |
| **How** | 一个文件夹 + 一个 `SKILL.md` + 可选脚本；通过 Slash 指令、`description` 自动路由或 Skill 嵌套触发；与 Claude Code、QoderWork、Cursor 等多家宿主互通。 |
| **Who Owns** | 业务专家提供 SOP，平台/工程团队封装与治理；售前可现场演示一个 Skill 在 5 分钟内被创建并执行。 |

---

## 目录

- [Skill 深度解读：售前内部培训手册](#skill-深度解读售前内部培训手册)
  - [30 秒电梯演讲（售前可直接复用）](#30-秒电梯演讲售前可直接复用)
  - [目录](#目录)
  - [1. Skill 是什么（What）](#1-skill-是什么what)
    - [1.1 三句话定义](#11-三句话定义)
    - [1.2 Skill 在技术栈中的位置](#12-skill-在技术栈中的位置)
    - [1.3 Skill 的解剖结构](#13-skill-的解剖结构)
  - [2. 为什么需要 Skill（Why）](#2-为什么需要-skillwhy)
    - [2.1 三问决策启发法](#21-三问决策启发法)
    - [2.2 Skill vs. Tool vs. Agent：横向对比](#22-skill-vs-tool-vs-agent横向对比)
    - [2.3 客户价值四问：售前现场常用论据](#23-客户价值四问售前现场常用论据)
    - [2.4 "使用 Skill" 的具体触发信号](#24-使用-skill-的具体触发信号)
  - [3. Skill 怎么落地（How，含 `pptx-reader` 案例）](#3-skill-怎么落地how含-pptx-reader-案例)
    - [3.1 为什么 PPTX 提取适合做成 Skill](#31-为什么-pptx-提取适合做成-skill)
    - [3.2 目录结构](#32-目录结构)
    - [3.3 `SKILL.md` 前言（Front Matter）拆解](#33-skillmd-前言front-matter拆解)
    - [3.4 把工作流当作状态机来看](#34-把工作流当作状态机来看)
    - [3.5 为什么这是 Skill 而非 Agent](#35-为什么这是-skill-而非-agent)
  - [4. Claude Code 内置 Skill](#4-claude-code-内置-skill)
    - [4.1 `anthropics/skills` 仓库](#41-anthropicsskills-仓库)
    - [4.2 常见的内置 Skill](#42-常见的内置-skill)
    - [4.3 内置 Slash 指令 vs. Skill](#43-内置-slash-指令-vs-skill)
  - [5. QoderWork 内置 Skill](#5-qoderwork-内置-skill)
    - [5.1 分类目录](#51-分类目录)
    - [5.2 QoderWork Skill 的调用方式](#52-qoderwork-skill-的调用方式)
  - [6. 常见 Agent 的 Skill 全景图（售前演示库）](#6-常见-agent-的-skill-全景图售前演示库)
    - [6.1 七大业务场景对应表](#61-七大业务场景对应表)
    - [6.2 高价值演示组合（5 分钟可演完）](#62-高价值演示组合5-分钟可演完)
  - [7. 设计自定义 Skill 的检查清单](#7-设计自定义-skill-的检查清单)
  - [8. 反模式回顾](#8-反模式回顾)
  - [9. 售前常见问题应答（FAQ）](#9-售前常见问题应答faq)
  - [10. 总结](#10-总结)

---

## 1. Skill 是什么（What）

### 1.1 三句话定义

1. **Skill** 是封装好的*程序性知识*单元——"给定输入 X，按以下顺序执行这些步骤，产出 Y"。
2. 它**不是** Tool（Skill 编排多个 Tool），也**不是** Agent（Skill 走预定流程，而不是从零决定下一步该做什么）。
3. 在落地形态上，Skill 通常是一个文件夹，包含一个 `SKILL.md`（提示词 + 元数据 + 步骤说明）以及可选的辅助脚本／资源。

与 [`vibe-coding-intro-for-traditional-dev.md` §1.4](./vibe-coding-intro-for-traditional-dev.md#14-skill) 中的既有定义相互印证：

> Skill 是一组固定的 SOP（标准操作流程），由**提示词 + 状态机 + 工作流**表达。Skill 不需要 LLM 临场推理出新步骤；它*可以*在某些步骤中调用 LLM，但流程本身是确定性的。

### 1.2 Skill 在技术栈中的位置

```text
┌─────────────────────────────────────────────────┐
│ Agent 层    "决定下一步做什么"                    │  ← LLM 推理循环
├─────────────────────────────────────────────────┤
│ Skill 层    "按固定流程跑一遍"                    │  ← 提示词 + 状态机
├─────────────────────────────────────────────────┤
│ Tool 层     "执行一个原子能力"                    │  ← 函数 / MCP Server
└─────────────────────────────────────────────────┘
```

**成本梯度**：Tool ≪ Skill < Agent。引言文档第 1 节给出的设计原则依然适用：*"能用 Tool 不用 Skill，能用 Skill 不用 Agent。"*

### 1.3 Skill 的解剖结构

一个 Skill 由三种要素组合而成：

| 要素 | 作用 | 典型载体 |
| :--- | :--- | :------- |
| **提示词（Prompt）** | 声明角色、范围、约束与触发条件 | `SKILL.md` 正文 |
| **状态机（State machine）** | 命名每个步骤及其转换条件 | `SKILL.md` 中的编号小节，或一段轻量 DSL |
| **工作流（Workflow）** | Tool / 脚本 / LLM 调用的具体执行顺序 | Bash 片段、Python 辅助脚本或 MCP 调用 |

Anthropic 使用（QoderWork 沿用）的目录约定为：

```text
my-skill/
├── SKILL.md            # 必需 —— 前言 + 流程
├── scripts/            # 可选辅助脚本
├── assets/             # 可选静态资源（模板、图标、提示词等）
└── references/         # 可选的供提示词引用的参考文档
```

`SKILL.md` 顶部的 **前言（front matter）** 是运行时用来索引 Skill 的关键：

```yaml
---
name: pptx-reader
description: "Read and analyze .pptx files. Invoke when the user wants to extract slide text or analyze a presentation."
---
```

`name` 是调用键；`description` 则是宿主模型用来判断"此次是否需要加载该 Skill"的*路由提示词*。

---

## 2. 为什么需要 Skill（Why）

### 2.1 三问决策启发法

复用 [`vibe-coding-intro-for-traditional-dev.md` §3.3](./vibe-coding-intro-for-traditional-dev.md#33-三问决策启发法) 的启发法，按代价从低到高依次提问：

| # | 问题 | 是 → 落到这里 | 例子 |
| :- | :--- | :----------- | :--- |
| Q1 | 能否用确定性、可验证的函数解决？ | **Tool** | 价格查询、哈希计算 |
| Q2 | 是否是固定的多步流程，只需要被编排？ | **Skill** | 结算流程、PPTX 提取流水线 |
| Q3 | 是否需要在新颖情况下决定下一步？ | **Agent** | 交易策略、代码重构规划 |

**当且仅当满足以下条件时**才把问题交给 Skill：问题是多步的、步骤可提前枚举，并且至少有一步能从自然语言指引中受益（让 LLM 自适应处理排版、摘要、错误措辞等）。

### 2.2 Skill vs. Tool vs. Agent：横向对比

| 维度 | Tool | Skill | Agent |
| :--- | :--- | :---- | :---- |
| 粒度 | 单一原子能力 | 多步流程 | 开放式目标 |
| LLM 参与度 | 通常无 | 可选，限定到具体步骤 | 处于规划循环的中心 |
| 状态 | 无状态 | 有流程状态（"当前在第 3 步"） | 含记忆 + 规划状态 |
| 失败行为 | 返回错误 | 进入预定义分支 / 回滚 | 重新规划，可能无限循环 |
| 调用方式 | RPC / 函数调用 | Slash 指令、`description` 自动路由或程序调用 | 长时会话 |
| 例子 | `extract_text(pptx)` | `pptx-reader`（提取 → 渲染 → 解包 → 摘要） | "调研这份幻灯片并写一份简报" |

### 2.3 客户价值四问：售前现场常用论据

客户最常问的四个问题及推荐应答框架——售前现场可逐题对答：

| 客户疑问 | 售前应答要点 | 关键论据 |
| :------- | :----------- | :------- |
| **"我已经买了 Copilot，为什么还要 Skill？"** | Copilot 是单行补全，Skill 是端到端 SOP 封装；两者互补不替代。 | Skill 一次封装跨越多个 Tool 的固定流程，把团队最佳实践沉淀为可分发资产。 |
| **"Skill 与提示词工程的差别？"** | 提示词工程是"写得好"，Skill 是"封装好 + 治理好"。 | Skill 自带版本、审计、回滚、Slash 触发；提示词只是 Skill 的一个组件。 |
| **"团队需要懂 Python 才能写 Skill 吗？"** | 不需要——`SKILL.md` 是 Markdown，业务专家也能写；只有需要外部工具时才写脚本。 | 最简 Skill 仅 20 行 Markdown；Qoder/Claude 都内置 `create-skill` 引导。 |
| **"Skill 会不会和我现有微服务/CLI 冲突？"** | 不冲突——Skill 编排现有工具，不替换它们；MCP Server 把已有能力暴露给 Skill 即可。 | 完全沿用客户存量资产，零迁移成本，仅增量加一层编排。 |

### 2.4 "使用 Skill" 的具体触发信号

当你在团队中观察到下列任一模式时，就该考虑沉淀为 Skill：

1. **同样的提示词 + 步骤被反复粘贴**——封装一次，之后按名调用即可。
2. **一个流程跨越多个 Tool**（例如：下载 → 解析 → 转换 → 摘要），且顺序很重要。
3. **新人上手成本高**——Skill 把"我们这里 X 任务的标准做法"显式编码，让新成员（与 AI）按同样 SOP 执行。
4. **流程要确定、措辞要灵活**——例如翻译 Skill、文档评审 Skill。
5. **你想要 Slash 指令的交互体验**——一键触发已知流程。

反过来，**不要**把以下情况包装成 Skill：

- 整个任务就是一次函数调用 → 保留为 Tool。
- 每次执行都需要全新的规划 → 使用 Agent。
- 所谓的 "Skill" 实际只是一段自由发挥的 LLM 提示词、没有真实流程 → 它只是一段 system prompt。

---

## 3. Skill 怎么落地（How，含 `pptx-reader` 案例）

本节以一个真实 Skill —— 位于 [`/Users/wangtianqing/Project/skills/awesome-skills/skills/pptx-reader`](/Users/wangtianqing/Project/skills/awesome-skills/skills/pptx-reader) 的 `pptx-reader`——把上述概念落到实处。

### 3.1 为什么 PPTX 提取适合做成 Skill

从 `.pptx` 文件中抽取内容*不是*一次函数调用就能搞定的：

1. PPTX 是压缩的 XML 包——纯文本需要解析器。
2. 部分幻灯片的核心信息靠**版式与图片**承载，纯文本提取会丢失含义。
3. 一个有用的流水线通常需要**三类交付物**：纯文本（用于 grep / RAG）、缩略图网格（供人快速浏览）、逐页 JPEG（供多模态 LLM 分析）。
4. 所需的 CLI（`markitdown`、`LibreOffice`、`pdftoppm`）位于 Python 之外，需要独立环境。

每一步都是确定性的，但*组合*的粒度对单个 Tool 太粗、对 Agent 又太死板——这正是 Skill 的甜区。

### 3.2 目录结构

```text
pptx-reader/
├── SKILL.md                 # 85 行 —— 提示词 + 5 节流程
└── scripts/
    ├── __init__.py
    ├── thumbnail.py         # 生成多页概览图
    ├── office/
    │   ├── soffice.py       # 无头 LibreOffice 包装器（PPTX → PDF）
    │   └── unpack.py        # 把 PPTX 压缩包解开为原始 XML 树
    └── venv/                # 隔离的 Python 环境
```

`SKILL.md` 是*规范*，`scripts/` 文件夹是*实现*。运行时先读取 `SKILL.md`，只有当提示词指示时才执行脚本。

### 3.3 `SKILL.md` 前言（Front Matter）拆解

```yaml
---
name: pptx-reader
description: "理解、读取和分析 .pptx 幻灯片文件内容。当用户需要提取 PPT 文本或分析演示文稿时调用此技能。"
---
```

- **`name: pptx-reader`** —— 作为调用键使用（`/pptx-reader` 或 `Skill(skill="pptx-reader")`）。
- **`description`** —— 驱动自动路由。宿主 LLM 会读取所有可用 Skill 的描述，当用户说出*"请帮我总结这份幻灯片"*时，挑选最匹配的那一个。

两条实操经验：

1. 描述**必须**写出触发条件（"当用户希望 X 时"）。否则路由器无从匹配。
2. 描述**不能**模糊（如"处理文档"）。路由器依赖具体关键词，例如 `.pptx`、`slide`、`presentation`。

### 3.4 把工作流当作状态机来看

`SKILL.md` 正文（第 1~5 节）实际上是一台状态机：

```text
[Start]
  │
  ├─► Step 1: 快速文本提取
  │     Tool: `python -m markitdown presentation.pptx > output.md`
  │     输出: markdown 文本
  │
  ├─► Step 2: 环境准备（仅首次运行）
  │     - 创建 venv
  │     - pip install "markitdown[pptx]" Pillow
  │
  ├─► Step 3: 深度提取（分支）
  │     ├── 3a: thumbnail.py        → 视觉概览网格
  │     └── 3b: office/unpack.py    → 原始 XML 树
  │
  ├─► Step 4: 转图像（需要多模态审阅时）
  │     - office/soffice.py PPTX → PDF
  │     - pdftoppm  PDF → JPEG 序列
  │
  └─► Step 5: 依赖检查 / 兜底
        - markitdown[pptx]、Pillow、LibreOffice、Poppler
```

对传统团队而言，几个值得指出的设计决策：

| 决策 | 为什么重要 |
| :--- | :--------- |
| **一条快速路径 + 多条深度路径** | 多数用户只需要纯文本；深度路径存在但默认不启用 |
| **隔离的 venv** | 规避 macOS 上 `externally-managed-environment` 的坑 |
| **统一的 `output_dir/`** | 可预期的文件布局让下游 Skill（如 `md-summarizer`）易于串联 |
| **Skill 内不调用 LLM** | 纯编排——是否对提取结果做加工，由调用方 Agent 决定 |

### 3.5 为什么这是 Skill 而非 Agent

套用三问启发法：

- Q1：*单个确定性函数？* 否——至少需要解析 + 渲染 + 解包。
- Q2：*固定流程？* 是——`text → thumbnail → unpack → image` 的顺序从未变化。
- Q3：*需要新颖决策？* 否——LLM 不需要发明新步骤。

→ **Skill 就是正确的层。** 如果硬把它做成 Agent，每次调用都要重推同一份计划；成本随 token 数增长，而非随实际工作量增长。

---

## 4. Claude Code 内置 Skill

### 4.1 `anthropics/skills` 仓库

Anthropic 在 [`anthropics/skills`](https://github.com/anthropics/skills) 维护了一组参考 Skill。Claude Code 可以从以下三处加载 Skill：

1. 上述官方仓库（由 Anthropic 维护）。
2. 用户级目录 `~/.claude/skills/`。
3. 项目级目录 `.claude/skills/`（这样团队就能把 Skill 和仓库一起交付）。

每个 Skill 都遵循 §1.3 描述的 `SKILL.md` + `scripts/` 结构。

### 4.2 常见的内置 Skill

下表的分类涵盖了大多数传统团队最先会接触到的模式：

| 类别 | 示例 Skill | 用途 |
| :--- | :-------- | :--- |
| **文档 I/O** | `pdf`、`pptx`、`docx`、`xlsx` | 读 / 写 / 提取 Office 系列文件 |
| **创作类** | `artifacts-builder`、`canvas-design` | 生成可视化制品（HTML、SVG、图表） |
| **工程类** | `mcp-builder`、`skill-creator`、`webapp-testing` | 引导生成 MCP Server、脚手架新 Skill、运行 Web 应用测试 |
| **仓库流程** | `pr-review`、`commit` | 引导标准的 Git / PR 礼仪 |

几条观察：

- Office 类 Skill（`pdf`、`pptx`……）的形态**与 `pptx-reader` 完全一致**——前言 + 编排外部 CLI 的流程。社区里有不少 Skill 是直接移植版本。
- `skill-creator` 本身就是一个用来写新 Skill 的 Skill——这是 Skill 生态常见的自举模式。
- Skill 是*热加载*的：在 `~/.claude/skills/` 下放入新文件夹，下次会话就能发现。

### 4.3 内置 Slash 指令 vs. Skill

注意一个常见误解（引言文档 §1.4.1 也提到过）：

| 类型 | 例子 | 定义位置 | 可自定义？ |
| :--- | :--- | :------- | :-------- |
| **内置 Slash 指令** | `/commit`、`/review-pr`、`/clear` | 硬编码在 Claude Code 二进制中 | ❌ |
| **用户自定义 Skill** | `/pdf`、`/opsx:propose`、`/pptx-reader` | Skill 目录下的 `SKILL.md` 文件 | ✅ |

判别口诀：能 `cat SKILL.md` 看到的就是 Skill；只活在编辑器源码里的，就是内置指令。

---

## 5. QoderWork 内置 Skill

> **辨明**：**Qoder** 是 IDE（编码 Agent），**QoderWork** 是 Qoder 团队推出的桌面 AI Agent（独立产品）。本节列出的 Skill 目录来自 QoderWork 桌面端运行时——Qoder IDE 与 QoderWork 共享同一套 Skill 框架（`SKILL.md` + `scripts/`），但默认装载的 Skill 集合不同。

QoderWork 的默认 Skill 目录比原生 Claude Code 更丰富，按用途大致分组。下面这份目录是当前 QoderWork 会话中实际可发现的 Skill 集合。

### 5.1 分类目录

| 类别 | Skill | 一句话用途 |
| :--- | :---- | :--------- |
| **Skill / Agent 元能力** | `create-skill`、`agent-skill-reviewer`、`create-subagent` | 编写并评审 Skill 与 subagent |
| **代码库理解** | `code-reader`、`project-analyzer` | 为陌生仓库构建认知地图 / 输出项目白皮书 |
| **文档 I/O** | `pptx-reader`、`md-summarizer`、`md-translator`、`md-link-checker`、`web-content-downloader`、`web-content-translator`、`web-summarizer` | 读取、翻译、总结、校验文档与网页 |
| **文档工程** | `doc-reviewer`、`tech-outline-planner`、`reference-organizer` | 多维度文档评审、提纲规划、引用整理 |
| **可视化制品** | `canvas`、`drawio-designer`、`editorial-card-designer` | 图表、信息卡、draw.io 流程图 |
| **流程 / 治理** | `openspec-assistant`、`update-submitter`、`peer-review`、`patent-application-review` | 规范驱动开发、约定式提交、同行 / 专利评审 |
| **DevOps 集成** | `vercel-deploy` | 一键部署 Web |
| **知识图谱** | `ontology` | 跨 Skill 的结构化 Agent 记忆 |
| **文件治理** | `dir-organizer` | 整理 / 清理项目目录 |

### 5.2 QoderWork Skill 的调用方式

调用方式有三种，与引言文档 §1.4.5 一致：

1. **直接 Slash 指令** —— 用户输入 `/pptx-reader path/to/deck.pptx`。
2. **基于 description 的自动路由** —— 用户说*"帮我总结这份幻灯片"*，宿主模型根据 `description` 选中 `pptx-reader`。
3. **Skill 调用 Skill** —— `project-analyzer` 内部可能调用 `code-reader` 与 `md-summarizer` 来组装它的白皮书。

对传统团队来说，落地路径很短：

```text
1. 列出可用 Skill（QoderWork 会自动展示）。
2. 挑一个描述与日常重复任务匹配的 Skill。
3. 用 /<name> 触发一次。
4. 如果团队特有的重复流程缺位，运行 /create-skill 脚手架一个新的 Skill。
```

---

## 6. 常见 Agent 的 Skill 全景图（售前演示库）

下面这份清单覆盖了在 Claude Code、QoderWork、Cursor 等主流 Agent 中**真实存在且高频使用**的 Skill 类别，按业务场景归类——售前可按客户行业直接挑选演示组合。

### 6.1 七大业务场景对应表

| 场景 | 代表 Skill | 解决的业务痛点 | 客户演示一句话 |
| :--- | :--------- | :------------- | :------------- |
| **办公文档处理** | `pptx-reader`、`pdf`、`docx`、`xlsx` | 把客户的 PPT / 合同 / Excel 转成 LLM 可消费的结构化文本 | "把这份 50 页招标书读进来，10 秒后给我关键条款摘要。" |
| **代码与工程治理** | `code-reader`、`project-analyzer`、`commit`、`pr-review`、`update-submitter` | 新人快速读懂陌生代码库；自动生成规范的提交与 PR | "拉一个陌生开源仓库，5 分钟产出架构白皮书。" |
| **规范驱动开发** | `openspec-assistant`、`@ddd-*`、`create-subagent` | 把 PRD 落到可验证的 Spec，再到代码与测试 | "从访谈记录到 OpenSpec 变更集，端到端可重放。" |
| **创作与设计** | `editorial-card-designer`、`drawio-designer`、`canvas`、`tech-outline-planner` | 一键生成发布会信息卡、架构图、技术文章提纲 | "把这段产品文案变成 16:9 的封面卡。" |
| **内容采集与翻译** | `web-content-downloader`、`web-content-translator`、`web-summarizer`、`md-translator`、`md-summarizer`、`md-link-checker` | 海外资讯 / 竞品文档的下载、翻译、摘要、链接体检一站完成 | "把这篇英文博客抓下来，翻成中文并提取图片。" |
| **DevOps 与发布** | `vercel-deploy`、`update-submitter` | 一键部署演示站点；按规范自动分组提交 | "改完代码后一句话部署到 Vercel 并产出 Conventional Commit。" |
| **元能力（Skill 工厂）** | `create-skill`、`agent-skill-reviewer`、`create-subagent`、`ontology` | 客户自己持续生产并治理 Skill | "现场为客户写一个新 Skill，并通过 reviewer 验收。" |

### 6.2 高价值演示组合（5 分钟可演完）

| 客户类型 | 推荐演示链路 | 用到的 Skill |
| :------- | :----------- | :----------- |
| **传统软件团队** | 拉陌生开源项目 → 出白皮书 → 自动生成 commit | `code-reader` → `project-analyzer` → `update-submitter` |
| **咨询 / 研究类客户** | 抓海外博客 → 翻译 → 总结 → 出信息卡 | `web-content-translator` → `md-summarizer` → `editorial-card-designer` |
| **产品经理 / PMO** | 访谈记录 → DDD → OpenSpec | `@ddd-*` → `openspec-assistant` |
| **架构师 / CTO 办公室** | PPT 招标书读取 → 关键条款摘要 → 架构图草稿 | `pptx-reader` → `md-summarizer` → `drawio-designer` |
| **专利 / 法务相关** | 交底书审阅 → 同行评审清单 → 引用规范化 | `patent-application-review` → `peer-review` → `reference-organizer` |

> **演示要点**：每个组合都遵守"一句自然语言指令 → 三步 Skill 链 → 一份可交付物"的节奏，便于客户在 5 分钟内看到端到端价值。

---

## 7. 设计自定义 Skill 的检查清单

撰写新的 `SKILL.md` 时，按下面这份清单走一遍：

- [ ] **触发明确**：`description` 是否写出了具体的用户意图，并至少包含一个路由器可匹配的关键词？
- [ ] **步骤固定**：能否在没有歧义的前提下给步骤编号？如果不行，它大概率应该是 Agent 而不是 Skill。
- [ ] **委托 Tool**：每个确定性步骤都调用 Tool / 脚本——而不是让 LLM 帮你算数。
- [ ] **状态边界**：每一步都有清晰的前置条件与后置条件（让失败可被局部化）。
- [ ] **幂等性**：在相同输入下重复运行 Skill 安全无副作用。
- [ ] **回滚 / 兜底**：至少有一条显式分支处理"上一步失败"。
- [ ] **输出契约**：交付物（路径、格式）有文档化说明，下游 Skill / Agent 能据此串联。
- [ ] **不在提示词中放秘密**：API Key 与凭证写在环境变量里，不写进 `SKILL.md`。
- [ ] **可审计**：日志 / 输出落到约定位置，便于复核。

---

## 8. 反模式回顾

下列五个反模式来自引言文档 §1.4.6，本节多加了一列——*在代码评审中如何识别*。

| 反模式 | 在 `SKILL.md` 中的症状 | 评审识别信号 |
| :----- | :-------------------- | :----------- |
| **Skill 承载物理 / 数值计算** | 提示词里写"请计算……"而不是调用脚本 | LLM 输出里出现的数字，调用方代码从未做合理性校验 |
| **Skill 替代 Agent** | 在固定步骤中出现"决定是否……" | 同样的输入跨次运行输出会不可预测地分叉 |
| **Skill 变大杂烩** | 30+ 步，混杂推理、计算、通知 | `SKILL.md` 长度超过 500 行，`description` 里出现多个不相关关键词 |
| **Skill 直接调用敏感 API** | 提示词或脚本里出现 `curl https://prod-db/...` | 完全没有提到鉴权 / 幂等 / 审计 |
| **Skill 缺少回滚** | 流程结尾写"失败就抛出异常" | 状态机中没有对应的补偿步骤 |

---

## 9. 售前常见问题应答（FAQ）

| 问题（客户/CTO 视角） | 应答要点 |
| :-------------------- | :------- |
| **Skill 是 Anthropic 专属吗？** | 不是。Anthropic 在 `anthropics/skills` 维护参考集，但 `SKILL.md` 是开放约定；Claude Code、QoderWork、Cursor 都能加载，宿主之间可互通。 |
| **私有 Skill 会被上传到云端吗？** | 默认不上传。Skill 文件位于本地仓库或用户目录，调用时仅元数据进入路由器，敏感凭证写入环境变量而非 `SKILL.md`。 |
| **Skill 的运行成本如何核算？** | Skill 本身近乎零成本，主要消耗来自其内部调用的 Tool 与 LLM；同等任务下 token 比纯 Agent 方案低 60–80%。 |
| **Skill 是否需要定期重训练？** | 不需要训练。Skill 是规则化资产，更新即热生效；只有当其调用的底层 LLM 升级时才需回归测试。 |
| **能否给客户 SLA？** | 能——Skill 流程可枚举，每步均可加超时 / 重试 / 回滚。建议以"步骤级 SLA"承诺，不对 LLM 输出做强 SLA。 |
| **Skill 与 LangChain / AutoGen 的关系？** | 它们是开发框架，Skill 是产物形态。可结合：用框架实现，用 `SKILL.md` 形式分发与治理。 |
| **客户最快多久能上线第一个 Skill？** | 当天。模板化 Skill（文档摘要、PR 评审）30 分钟可定制完毕；复杂业务 Skill 1–2 周。 |
| **Skill 出错怎么办？** | 状态机显式定义补偿步骤；流程级日志可审计；必要时由调用方 Agent 接管重规划。 |

---

## 10. 总结

Skill 是把"零散粘贴的提示词"变成"可复用、可审计 SOP"的**中间层**。它在技术栈中占有一席之地的前提是：

1. 工作是多步的；
2. 步骤可提前枚举；
3. 至少有一步能从自然语言的灵活性中受益。

回看本文中的三组参考点：

- `pptx-reader` Skill 用 85 行 `SKILL.md` 加一个小型 `scripts/` 文件夹，演示了规范的**提示词 + 状态机 + 工作流**模式。
- Claude Code 内置 Skill（`pdf`、`pptx`、`skill-creator`……）提供了基础目录，并展示了"用 Skill 写 Skill"的自举模式。
- QoderWork 内置目录更广（24+ 个 Skill，覆盖创作、评审、部署与元能力），还把"Skill 调用 Skill"的组合能力开箱即用。

给售前团队的核心建议：**面对客户时按四步走——讲价值（§2.3 客户价值四问）→ 演场景（§6 演示库）→ 当场创建 Skill（`/create-skill`）→ 用 FAQ（§9）收尾**。当客户的 Skill 目录不断扩展，Skill 层就成为他们外化的程序性记忆——可读、可版本管理，并且对人和 AI 同样可用。

**售前"招式包"四件套**：30 秒电梯演讲（开篇）+ 客户价值四问（§2.3）+ 七大场景演示库（§6）+ FAQ（§9）。

> 延伸阅读：[`vibe-coding-intro-for-traditional-dev.md`](./vibe-coding-intro-for-traditional-dev.md)（基础概念）、[`anthropics/skills`](https://github.com/anthropics/skills)（参考 Skill 集）、[`/Users/wangtianqing/Project/skills/awesome-skills/skills/pptx-reader/SKILL.md`](/Users/wangtianqing/Project/skills/awesome-skills/skills/pptx-reader/SKILL.md)（本文案例）。
