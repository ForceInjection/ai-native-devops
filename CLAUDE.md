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

`.claude/settings.local.json` has allow-listed permissions for this repo: `python3`, `git add/rm/commit`, `WebSearch`, and several `WebFetch` domains (Wikipedia, Baidu, Google). When modifying settings, update this file rather than creating new config locations.

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

**Relation to CloudPilot**: CloudPilot now covers P1-pre → P6(代码桥接) → P7(实现工作流，Stage A→E 生成可运行代码)。OpenSpec-practise 补充了多语言实现(Node.js+Python)和部署/验证阶段。两者加在一起形成完整的 8 阶段 walkthrough。

## Skill 与 Agent 清单

`/cloudpilot-demo` 全链路涉及 22 个 Skill/Agent，分为已实现和待实现两类。完整清单和功能说明见 [`cloudpilot-case/README.md` §Skill 清单](./cloudpilot-case/README.md#skill-清单)。

### 已实现（13 个）

| 类型 | 名称 | 说明 |
| :--- | :--- | :--- |
| Subagent | `ddd-modeler` | 串行驱动 9 个 `@ddd-*` Skill，质量门禁 <80% 回溯 |
| Subagent | `openspec-author` | 将 DDD 模型转为完整的 OpenSpec 变更集 |
| Skill (×9) | `@ddd-scope` ~ `@ddd-openspec-bridge` | DDD 领域建模全流程（发现→战略→战术→验证→桥接） |
| Skill | `openspec-assistant` | `/opsx:*` 命令体系（propose/apply/verify/archive） |
| Skill | `open-code-review` | AI 代码评审（`ocr` CLI），检查代码与需求匹配 |

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
- All generated markdown files for the case study include a frontmatter block:

  ```text
  阶段: <P1-P8 phase name>
  上游输入: <source artifact file>
  下游消费: <next phase artifact>
  责任人: <human owner role>
  AI 草稿置信度: <percentage>
  ```

- Markdown cross-references between sibling files use repo-relative paths (e.g. `[../03-ddd-modeling.md](../03-ddd-modeling.md)`) rather than absolute paths or bare filenames.
