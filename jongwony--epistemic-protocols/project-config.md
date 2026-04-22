---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Epistemic Protocols is a layered system for human-AI collaboration: it inserts structured checkpoints at decision points so misalignment is surfaced early, judged explicitly, and adapted before it compounds into expensive downstream work.

In this repository, that machinery is realized as a Claude Code plugin marketplace for epistemic dialogue — each protocol structures a specific decision point: **FrameworkAbsent → FramedInquiry** (Prothesis), **GapUnnoticed → AuditedDecision** (Syneidesis), **IntentMisarticulated → ClarifiedIntent** (Hermeneia), **ResultUngrasped → VerifiedUnderstanding** (Katalepsis), **GoalIndeterminate → DefinedEndState** (Telos), **BoundaryUndefined → DefinedBoundary** (Horismos), **ContextInsufficient → InformedExecution** (Aitesis), **MappingUncertain → ValidatedMapping** (Analogia), **ExecutionBlind → SituatedExecution** (Prosoche), **ApplicationDecontextualized → ContextualizedExecution** (Epharmoge), **RecallAmbiguous → RecalledContext** (Anamnesis) during human-AI interaction.

## Architecture

```
epistemic-protocols/
├── .claude-plugin/marketplace.json    # Marketplace manifest
├── .claude/skills/verify/             # Project-level verification skill
│
│   # Each protocol plugin: .claude-plugin/plugin.json + skills/<verb>/SKILL.md
├── prothesis/       (/frame)          # multi-perspective investigation
├── syneidesis/      (/gap)            # gap surfacing
├── hermeneia/       (/clarify)        # intent clarification
├── katalepsis/      (/grasp)          # comprehension verification
├── telos/           (/goal)           # goal co-construction
├── horismos/        (/bound)          # epistemic boundary definition
├── aitesis/         (/inquire)        # context insufficiency inference
├── analogia/        (/ground)         # structural mapping validation
├── prosoche/        (/attend)         # execution-time risk evaluation
├── epharmoge/       (/contextualize)  # application-context mismatch (conditional)
├── anamnesis/       (/recollect)      # vague recall → recognized context
│   ├── hooks/hooks.json               # SessionEnd hook: hypomnesis store writer
│   └── scripts/hypomnesis-write.mjs   # mjs harness + claude -p haiku extraction
├── epistemic-cooperative/             # Utility skills + agents
│   ├── agents/                        # project-scanner, session-analyzer, coverage-scanner, dimension-profiler
│   └── skills/                        # report, onboard, dashboard, introspect, catalog, compose, sophia, curses, write, artifact-review, review-ensemble
└── src/                               # Landing page (independent sub-project; React + Vite + Tailwind; EN/KO SPA)
```

**Component Types**:
- **Skills** (`skills/*/SKILL.md`): Full protocol/workflow definitions with YAML frontmatter; user-invocable by default (v2.1.0+)
- **Agents** (`agents/*.md`): Subagents for parallel task execution (epistemic-cooperative)

**Conventions**:
- Subagent naming: `plugin-name:agent-name` (e.g., `epistemic-cooperative:session-analyzer`)
- References directory: `skills/*/references/` for detailed documentation (optional per plugin)
- No external dependencies; Node.js standard library only (plugin code). `src/` landing page is an independent sub-project with its own `package.json`

**Plugin Encapsulation**: Runtime users interact with the packaged runtime contract: `Skill.md` (normative user contract) plus plugin description metadata (discovery/routing only, not full semantics). `.claude/rules/` prescriptive changes affecting protocol behavior must be compiled into `Skill.md` Rules sections. `Skill.md` must be self-contained — no external references (axiom identifiers, rule file paths, design-philosophy concepts, mission/vision docs) that require reading contributor documentation. Claim-strength boundaries for each runtime surface are tracked in [docs/runtime-dependency-ledger.md](docs/runtime-dependency-ledger.md).

**SKILL.md Formal Block Anatomy**: FLOW, MORPHISM, TYPES, PHASE TRANSITIONS, LOOP, TOOL GROUNDING, ELIDABLE CHECKPOINTS, MODE STATE, COMPOSITION (and optional blocks). Details: [docs/structural-specs.md](docs/structural-specs.md#skillmd-formal-block-anatomy)

## Plugins

| Protocol | Slash | Deficit → Resolution |
|----------|-------|----------------------|
| Prothesis | `/frame` | FrameworkAbsent → FramedInquiry |
| Syneidesis | `/gap` | GapUnnoticed → AuditedDecision |
| Hermeneia | `/clarify` | IntentMisarticulated → ClarifiedIntent |
| Katalepsis | `/grasp` | ResultUngrasped → VerifiedUnderstanding |
| Telos | `/goal` | GoalIndeterminate → DefinedEndState |
| Horismos | `/bound` | BoundaryUndefined → DefinedBoundary |
| Aitesis | `/inquire` | ContextInsufficient → InformedExecution |
| Analogia | `/ground` | MappingUncertain → ValidatedMapping |
| Prosoche | `/attend` | ExecutionBlind → SituatedExecution |
| Epharmoge | `/contextualize` | ApplicationDecontextualized → ContextualizedExecution |
| Anamnesis | `/recollect` | RecallAmbiguous → RecalledContext |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jongwony/epistemic-protocols](https://github.com/jongwony/epistemic-protocols) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
