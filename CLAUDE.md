# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository purpose

This is a whitepaper repository proposing an "AI Native DevOps" framework — a DevOps collaboration model where AI serves as an enhancer (not a replacement) across requirements, design, modeling, specification, implementation, verification, delivery, and evolution phases. Decision authority, risk accountability, and cross-team arbitration remain with explicit human owners.

## Key documents

- `ai-native-devops.md` — The main article (~54K, 712 lines). Defines the 8-phase framework, AI participation levels, maturity assessment, governance metrics, and implementation roadmap.
- `ai-native-devops-sample-change-walkthrough.md` — A companion walkthrough using an "order cancellation" scenario to demonstrate the full AI-assisted change lifecycle end-to-end.
- `ai-native-devops-panorama.html` / `.png` — Framework overview diagram (interactive HTML + static PNG).

## Reference projects

The article's "existing capability" claims are grounded in two public reference projects (not part of this repo):
- `domain-driven-design-skills` — 9 DDD Skills covering strategic/tactical modeling and OpenSpec bridging.
- `OpenSpec-practise` — Spec-driven development workflow with `proposal.md`, `design.md`, `tasks.md`, `specs/`, and `/opsx:*` commands.

## Conventions

- Diagrams use Mermaid (`flowchart TD`) embedded in markdown code blocks.
- Stage maturity is color-coded: green for existing capabilities, yellow for planned/gap areas.
- Every AI participation stage must explicitly note: AI input, AI output, suggested artifacts, and human confirmation points.
- Key artifacts (PRD, domain model, OpenSpec, deployment decisions) only proceed to the next phase after human sign-off.
- The framework treats `docs/`, `openspec/`, and engineering code as the single source of truth — conversation context must not diverge from repo state.
