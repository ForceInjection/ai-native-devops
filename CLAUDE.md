# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository purpose

This is a whitepaper repository proposing an "AI Native DevOps" framework — a DevOps collaboration model where AI serves as an enhancer (not a replacement) across requirements, design, modeling, specification, implementation, verification, delivery, and evolution phases. Decision authority, risk accountability, and cross-team arbitration remain with explicit human owners.

## Document map

```
vibe-coding-intro-for-traditional-dev.md   ← Entry point for traditional dev teams (concept primer)
├── ai-native-devops/                      ← Core framework
│   ├── ai-native-devops.md                (main article: 8-phase framework, governance, roadmap)
│   ├── ai-native-devops-sample-change-walkthrough.md  (order-cancellation end-to-end example)
│   ├── ai-native-devops-panorama.html     (interactive framework overview diagram)
│   ├── ai-native-devops-panorama.png      (static export of the panorama)
│   └── ai-native-devops.png               (static framework overview image)
├── ai-native-architecture/                ← AI Native Architecture
│   ├── ai-native-architecture.md          (power-trading case study → 3-layer pyramid derivation)
│   ├── ai-native-architecture-diagram.html (interactive pyramid: clickable Agent/Skill/Tool layers)
│   └── ai-native-architecture-diagram-article.md  (design rationale behind the diagram)
└── cloudpilot-case/                       ← Full 8-phase walkthrough (CloudPilot MVP)
    ├── README.md                          (index + reproducible prompts for all 5 phases)
    ├── 01-interview-notes.md              (P1-preq: business interview synthesis)
    ├── 02-prd.md                          (P1: structured PRD)
    ├── cloudpilot-mockup.html             (P2: interactive mock UI, 5 views, localStorage)
    ├── 03-ddd-modeling.md                 (P3: 9-skill DDD pipeline output)
    └── 04-openspec/                       (P4: proposal, design, tasks, 3 capability specs)
        ├── README.md                      (index + DDD→OpenSpec mapping table)
        ├── proposal.md                    (§Why / §What Changes / §Impact)
        ├── design.md                      (architecture, integration, key decisions)
        ├── tasks.md                       (staged implementation breakdown)
        └── specs/
            ├── billing/spec.md            (billing requirements + scenarios)
            ├── resource-management/spec.md
            └── resource-request/spec.md   (resource-request requirements + scenarios)
```

`reference/GBT+42560-2023/` contains a Chinese national standard for AI DevOps capability assessment — `full.md` is the extracted markdown, the PDF is the original standard document, and `images/` holds embedded figures.

Root-level `.pptx` files (`从 Vibe Coding 到 AI Native.pptx`, `ai-native-devops/` 下的 `AI Native DevOps：人机协同的工程变革框架.pptx`) are presentation slide decks derived from the articles; they are not source-of-truth documents.

## Reusable agent definitions

`.qoder/agents/` contains two subagent definitions used in the CloudPilot case study:

- **`ddd-modeler`** — Drives 9 `@ddd-*` skills end-to-end (I Discovery → II Strategic → III Tactical → IV Validation → V Spec bridge). Requires a PRD input, halts if `@ddd-model-review` score < 80%, emits a single consolidated DDD markdown.
- **`openspec-author`** — Turns a DDD model into a complete OpenSpec change set (proposal / design / tasks / per-context spec.md). Maps every `IV-N` invariant to at least one `Scenario:` block.

Both agents reference skills at `/Users/wangtianqing/Project/skills/domain-driven-design-skills/skills/` (external to this repo).

**Note:** `.qoder/` is listed in `.gitignore` and is not tracked in git. The agent definitions above are local-only artifacts used during the CloudPilot case study. When setting up a fresh clone, re-create them from the `cloudpilot-case/README.md` prompt templates if needed.

## Conventions

- Diagrams use Mermaid embedded in markdown code blocks (`flowchart TD`, `graph LR`, `sequenceDiagram`, `graph TB`).
- Stage maturity is color-coded: green for existing capabilities, yellow for planned/gap areas.
- Every AI participation stage must explicitly note: AI input, AI output, suggested artifacts, and human confirmation points.
- Key artifacts (PRD, domain model, OpenSpec, deployment decisions) only proceed to the next phase after human sign-off.
- The framework treats `docs/`, `openspec/`, and engineering code as the single source of truth — conversation context must not diverge from repo state.
- DDD aggregate invariants use stable `IV-N` numbering so downstream OpenSpec `Scenario:` blocks can cross-reference them.
- Capability/context names in OpenSpec use kebab-case (e.g., `resource-request`).
- All generated markdown files for the case study include a frontmatter block:

  ```
  阶段: <P1-P8 phase name>
  上游输入: <source artifact file>
  下游消费: <next phase artifact>
  责任人: <human owner role>
  AI 草稿置信度: <percentage>
  ```

- Markdown cross-references between sibling files use repo-relative paths (e.g. `[../03-ddd-modeling.md](../03-ddd-modeling.md)`) rather than absolute paths or bare filenames.
