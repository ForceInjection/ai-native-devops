# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository purpose

This is a whitepaper repository proposing a 3-layer "AI Native" engineering framework:

```text
Vibe Coding (个人) → AI Native DevOps (团队) → AI Native Architecture (应用架构)
```

- **Vibe Coding**: How individual developers use Agent/MCP/A2A/Skill to let AI produce drafts while humans review and gate.
- **AI Native DevOps**: An 8-phase framework where AI enhances (not replaces) requirements, design, modeling, specification, implementation, verification, delivery, and evolution. Decision authority, risk accountability, and cross-team arbitration remain with explicit human owners.
- **AI Native Architecture**: The Agent/Skill/Tool cost pyramid for embedding AI into product systems — "not everywhere needs an LLM."

The `README.md` serves as the human-readable 3-layer overview with role-based reading paths. This `CLAUDE.md` focuses on what Claude Code instances need to navigate and contribute to the repo.

## Document map

```text
README.md                                   ← 3-layer framework overview for human readers
vibe-coding-intro-for-traditional-dev.md   ← Entry point: Agent/MCP/A2A/Skill concept primer
├── skill-deep-dive-for-traditional-dev.md  (Skill deep-dive: what/when/how + built-in Skills catalog)
├── ai-native-devops/                      ← Core framework (team layer)
│   ├── ai-native-devops.md                (main article: 8-phase framework, governance, roadmap)
│   ├── sdd-paradigms-synthesis.md         (SDD 实践: AI Native DevOps 基于《SDD 实战》一书取长补短 — 范式盲区 → 对方取向 → 吸收设计 → openspec 工作区 change 生命周期实录)
│   ├── ai-native-devops-sample-change-walkthrough.md  (order-cancellation end-to-end example)
│   ├── ai-native-devops-panorama.html     (interactive framework overview diagram)
│   ├── ai-native-devops-panorama.png      (static export of the panorama)
│   └── ai-native-devops.png               (static framework overview image)
├── ai-native-architecture/                ← AI Native Architecture (application layer)
│   ├── ai-native-architecture.md          (power-trading case study → 3-layer pyramid derivation)
│   ├── ai-native-architecture-diagram.html (interactive pyramid: clickable Agent/Skill/Tool layers)
│   └── ai-native-architecture-diagram-article.md  (design rationale behind the diagram)
└── cloudpilot-case/                       ← Full 8-phase walkthrough (CloudPilot MVP)
    ├── README.md                          (index + reproducible prompts for all 8 phases)
    ├── 01-interview-notes.md              (P1-pre: business interview synthesis)
    ├── 02-prd.md                          (P1: structured PRD)
    ├── cloudpilot-mockup.html             (P2: interactive mock UI, 5 views, localStorage)
    ├── 03-ddd-modeling.md                 (P3: 9-skill DDD pipeline output)
    ├── 04-openspec/                       (P4: proposal, design, tasks, 3 capability specs)
    │   ├── README.md                      (index + DDD→OpenSpec mapping table)
    │   ├── proposal.md                    (§Why / §What Changes / §Impact)
    │   ├── design.md                      (architecture, integration, key decisions)
    │   ├── tasks.md                       (staged implementation breakdown)
    │   └── specs/
    │       ├── billing/spec.md            (billing requirements + scenarios)
    │       ├── resource-management/spec.md
    │       └── resource-request/spec.md   (resource-request requirements + scenarios)
    ├── config.yaml                        (AI context injection: schema, context, rules for proposal/specs/design/tasks/frontmatter/naming)
    ├── openspec/                          (CLI 实况工作区: init 生成, 主基线迁移自 04-openspec + contracts/ + adrs/ + changes/archive, Golden Set 副本只读源)
    │   ├── project.md                     (项目上下文: 数据契约/约定/已知 gap/术语表 — SDD project.md 惯例)
    │   ├── specs/{resource-request,resource-management,billing}/spec.md   (CLI 校验的主基线, billing 含 B-1~B-4 派生编号)
    │   ├── contracts/data-models.md       (常驻契约工件, 被 design/tasks 共同引用 — SDD book-code 惯例)
    │   ├── adrs/001-005-*.md              (常驻决策记录: D1-D4 提升 + IV-N/B-N 编号溯源规则)
    │   └── changes/archive/2026-08-05-enrich-specs-contracts-adrs/  (已归档 change: 契约+ADR+B-N 补齐)
    ├── 05-p5-code-bridge.md              (P6: 代码桥接 — spec→code mapping, contract design, Mock→Real switch)
    ├── 06-p5-implementation-workflow.md  (P7: 实现工作流 — 5-stage implementation pipeline with ocr)
    ├── cloudpilot-demo-nav.html          (interactive demo console: phases timeline + flow diagram + artifact preview modals)
    └── demo.cast                          (asciinema recording of the full demo walkthrough)
```

Root-level `.pptx` files (`从 Vibe Coding 到 AI Native.pptx`, `ai-native-devops/AI Native DevOps：人机协同的工程变革框架.pptx`) are presentation slide decks derived from the articles; they are not source-of-truth documents. Both are gitignored by the `*.pptx` pattern.

### Gitignored content

The `.gitignore` excludes these categories — don't attempt to commit or track them:

| Pattern                     | Reason                                                                                |
| :-------------------------- | :------------------------------------------------------------------------------------ |
| `.qoder/`                   | Local subagent definitions; recreate from `cloudpilot-case/README.md` prompts         |
| `.claude/skills/`           | Symlinked DDD skills + cloudpilot-demo skill; setup script in §Demo skill             |
| `reference/`                | Local reference materials (e.g., GBT+42560-2023 DevOps standard); not source-of-truth |
| `*.pptx`, `*.xlsx`, `*.gif` | Derived presentation files and large binaries; not source-of-truth                    |

### Reference directory

`reference/GBT+42560-2023/` contains the Chinese national standard **系统与软件工程 开发运维一体化 能力成熟度模型** (System and software engineering — DevOps — Capability maturity model, 2023-12-01 implementation). `full.md` is a machine-extracted full text of the standard. This standard informs the AI Native DevOps framework's maturity model but is not directly referenced in the articles.

## Local configuration

`.claude/settings.local.json` has allow-listed permissions for this repo: `python3`, `git add/rm/commit/push`, `WebSearch`, several `WebFetch` domains (Wikipedia, Baidu, Google, GitHub), plus the demo/P7 toolchain (`npm install/test/run`, `node`, `ocr review`, `asciinema`, `openspec`). When modifying settings, update this file rather than creating new config locations.

## 常用命令与工具链

本仓库是纯文档仓库（无构建/测试目标），常用操作集中在案例验证与演示：

| 操作                              | 命令                                                                                          |
| :-------------------------------- | :-------------------------------------------------------------------------------------------- |
| P4 验收：IV-N → Scenario 覆盖检查 | `grep -rE 'IV-[1-8]' cloudpilot-case/04-openspec/specs/`                                      |
| OpenSpec 结构校验 / 列清单        | `openspec spec validate <spec-id>` / `openspec list --specs`（`openspec` CLI，Homebrew 安装） |
| P7 生成代码的测试                 | 在输出目录（`$OUT/cloudpilot/`）下 `npm test` — 生成代码不入库                                |
| P7 OCR 代码评审                   | `ocr review`（alibaba/open-code-review CLI，Stage D 使用）                                    |
| 演示录像回放 / 重新录制           | `asciinema play cloudpilot-case/demo.cast`（录制用 `asciinema rec`）                          |
| DDD Skill 软链接                  | 见下方 §Demo skill 的 setup 脚本（新机器必做）                                                |

依赖的工具链：`claude` CLI、`openspec` CLI（`/opt/homebrew/bin/openspec`）、`ocr` CLI（`/opt/homebrew/bin/ocr` v1.0.6）、`asciinema`、9 个 `ddd-*` skill 软链接、`openspec-assistant` skill（`~/.claude/skills/openspec-assistant/`）。全部为本机安装，不随仓库分发。

## Reading paths by role

When recommending what to read, use these starting points (aligned with README.md):

| 角色                      | 入口                                                                                                                                  |
| :------------------------ | :------------------------------------------------------------------------------------------------------------------------------------ |
| **All readers**           | `vibe-coding-intro-for-traditional-dev.md` — establishes Agent/MCP/A2A/Skill terminology                                              |
| **PM**                    | `ai-native-devops/ai-native-devops.md` §1, §4.1, §7.6, §9.2, §10.4, §11.1                                                             |
| **Architect**             | `ai-native-devops/ai-native-devops.md` §4.3, §4.4, §7.2, §7.3, §7.8, §7.9 + `ai-native-architecture/ai-native-architecture.md` (full) |
| **Developer / Tech Lead** | `ai-native-devops/ai-native-devops.md` §4.5, §4.6, §6.1, §7.4, §7.7, §12 + `cloudpilot-case/`                                         |
| **Platform / SRE / QA**   | `ai-native-devops/ai-native-devops.md` §4.6, §4.7, §7.5, §7.8, §9, §11.3                                                              |
| **Demo 演示者**           | `.claude/skills/cloudpilot-demo/references/presenter-guide.md` — 10 分钟节奏、讲解要点、常见 Q&A                                      |

## Companion repositories

Two open-source repos complement the CloudPilot case study:

- **[ForceInjection/domain-driven-design-skills](https://github.com/ForceInjection/domain-driven-design-skills)** — 9 DDD skills (@ddd-scope through @ddd-openspec-bridge) used in P3 domain modeling.
- **[ForceInjection/OpenSpec-practise](https://github.com/ForceInjection/OpenSpec-practise)** — Full P4-P7 demo (e-commerce domain): same `proposal/design/tasks/specs` structure drives Node.js AND Python implementations from identical specs. Proves the spec format is language-agnostic. Includes `config.yaml` (AI context injection), archive workflow, and test patterns.
- **[huangjia2019/sdd-in-action](https://github.com/huangjia2019/sdd-in-action)** — 《SDD 实战》配套仓库 (book-code/ specs 含 contracts/+adrs/ 模板, 行动营 week1-4 学习路径). 融合与实践见 `ai-native-devops/sdd-paradigms-synthesis.md` 与 `cloudpilot-case/openspec/` 工作区；本地克隆在 `~/Project/skills/sdd-in-action` (不入库)。

**Relation to CloudPilot**: CloudPilot now covers P1-pre → P6(代码桥接) → P7(实现工作流，Stage A→E 生成可运行代码)。OpenSpec-practise 补充了多语言实现(Node.js+Python)和部署/验证阶段。sdd-in-action 补充方法论全集：proposal/design/tasks 之外增加 contracts/+adrs/ 模板（P4 增强）、brownfield 渐进引入（P8）与个人→团队→组织落地路径。三者加在一起形成完整的 8 阶段 walkthrough。

## Skill 与 Agent 清单

`/cloudpilot-demo` 全链路涉及 22 个 Skill/Agent，分为已实现和待实现两类。完整清单和功能说明见 [`cloudpilot-case/README.md` §Skill 清单](./cloudpilot-case/README.md#skill-清单)。

### 已实现（13 个）

| 类型       | 名称                                  | 说明                                               |
| :--------- | :------------------------------------ | :------------------------------------------------- |
| Subagent   | `ddd-modeler`                         | 串行驱动 9 个 `@ddd-*` Skill，质量门禁 <80% 回溯   |
| Subagent   | `openspec-author`                     | 将 DDD 模型转为完整的 OpenSpec 变更集              |
| Skill (×9) | `@ddd-scope` ~ `@ddd-openspec-bridge` | DDD 领域建模全流程（发现→战略→战术→验证→桥接）     |
| Skill      | `openspec-assistant`                  | `/opsx:*` 命令体系（propose/apply/verify/archive） |
| Skill      | `open-code-review`                    | AI 代码评审（`ocr` CLI），检查代码与需求匹配       |

### 待实现（9 个）

`interview-synthesizer`、`prd-generator`、`mockup-builder`、`coverage-checker`、`structure-deriver`、`code-generator`、`spec-validator`、`test-generator`、`archiver`。详见 README Skill 清单。

### 本地配置

`.qoder/agents/` contains the two subagent definitions. Both reference skills from [domain-driven-design-skills](https://github.com/ForceInjection/domain-driven-design-skills) (external open-source repo, cloned locally).

**Note:** `.qoder/` is listed in `.gitignore` and is not tracked in git. When setting up a fresh clone, re-create them from the `cloudpilot-case/README.md` prompt templates if needed.

## Demo skill

`.claude/skills/cloudpilot-demo/SKILL.md` defines a `/cloudpilot-demo` slash command that replays the full CloudPilot workflow end-to-end:

- Auto-detects current progress by checking which output files exist; supports `$OUT` directory switching via `输出到 <path>`
- Executes P1-pre → P1 → P2 → P3 → P4 → P5(验收) → P6(桥接) → P7(实现) sequentially, pausing for human confirmation at each stage
- P3 invokes 9 DDD skills (symlinked from `.claude/skills/ddd-*`) with quality gate at step 8; `快速 P3` skips per-skill display
- P4 generates `04-openspec/` with IV-N → Scenario coverage enforcement; `快速 P4` uses `openspec-assistant` skill's `/opsx:propose`
- P6: 代码桥接 generates `05-code-structure.md` — AI derives code structure from DDD model + specs in real-time, then `对比P6` compares against the pre-defined reference `cloudpilot-case/05-p5-code-bridge.md`
- P7: 实现 executes Stage A→E pipeline — Spec验证→测试先行(Red)→逐层实现(Green, contracts→domain→repo→services)→OCR评审→归档, producing runnable TypeScript code with full test coverage
- `对比` compares all P1-P4 outputs against `cloudpilot-case/` originals
- A presenter guide with talking points and common Q&A lives at `references/presenter-guide.md`

### cloudpilot-case 的双重角色（Golden Set）

`cloudpilot-case/` 下的文件承担两个角色：`01-interview-notes.md` 是 `/cloudpilot-demo` 的输入源（$SRC）；其余工件（`02-prd.md` 起）是「方法论正确执行后的参考产出物」，作为 `对比` / `对比P6` / `对比P7` 的**评估基准**。对比判据是**方法论一致性**——结构对齐、IV-N 覆盖完整、测试通过——而不是逐字相同：差异本身不是失败，但需要可解释。不要把这些 Golden 文件当作需要「修正」的代码，也不要让会话输出覆盖它们（输出请用 `输出到 <path>` 重定向到别处）。

`.claude/skills/` is gitignored. The DDD skills come from the open-source repo [ForceInjection/domain-driven-design-skills](https://github.com/ForceInjection/domain-driven-design-skills). On a new machine:

```bash
# 1. Clone the DDD skills repo
git clone https://github.com/ForceInjection/domain-driven-design-skills.git /path/to/ddd-skills

# 2. Symlink into this project
mkdir -p .claude/skills
for skill in ddd-scope ddd-discover ddd-subdomains ddd-contexts ddd-context-map \
              ddd-aggregates ddd-domain-interactions ddd-model-review ddd-openspec-bridge; do
  ln -sfn /path/to/ddd-skills/skills/$skill .claude/skills/$skill
done
```

## Conventions

- Diagrams use Mermaid embedded in markdown code blocks (`flowchart TD`, `graph LR`, `sequenceDiagram`, `graph TB`).
- Stage maturity is color-coded: green for existing capabilities, yellow for planned/gap areas.
- Every AI participation stage must explicitly note: AI input, AI output, suggested artifacts, and human confirmation points.
- Key artifacts (PRD, domain model, OpenSpec, deployment decisions) only proceed to the next phase after human sign-off.
- The framework treats `docs/`, `openspec/`, and engineering code as the single source of truth — conversation context must not diverge from repo state.
- DDD aggregate invariants use stable `IV-N` numbering so downstream OpenSpec `Scenario:` blocks can cross-reference them.
- Capability/context names in OpenSpec use kebab-case (e.g., `resource-request`).
- All generated markdown files for the case study open with a blockquote header directly after the `# NN · Title` line (not YAML frontmatter):

  ```text
  > **阶段**：<P1-P8 phase name>
  > **上游输入**：<source artifact file>（repo-relative link）
  > **下游消费**：<next phase artifact>
  > **责任人**：<human owner role>
  > **AI 草稿置信度**：<percentage>   (optional; specs/*/spec.md use **Capability**/**责任聚合** instead)
  ```

- Markdown cross-references between sibling files use repo-relative paths (e.g. `[../03-ddd-modeling.md](../03-ddd-modeling.md)`) rather than absolute paths or bare filenames.
