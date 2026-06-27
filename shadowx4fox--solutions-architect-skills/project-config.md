---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

> **Note**: This is a Claude Code Plugin. For plugin development documentation, see [Claude Code Plugin Development Guide](https://docs.anthropic.com/claude/docs/claude-code-plugins).

## Development Commands

```bash
# Type-check all TypeScript files
bun run typecheck

# Compile TypeScript (tsc)
bun run build

# Run unit tests (Bun test framework)
bun test

# Run tests in a specific file
bun test skills/architecture-compliance/tests/validators.test.ts

# Install workspace dependencies (root + tools/docgen)
bun install

# Build a release package (ZIP + SHA256)
bash scripts/build-release.sh
```

## Trigger Routing — Exact Phrases

When the user types any of these phrases, the correct FIRST action is to invoke the listed skill (not plan, not ask, not explore). These rules override plan-mode's default "Explore → Plan → ExitPlanMode" flow: skill invocation is a read-only context load that is legal inside plan mode, and the skill's own workflow handles clarification, state detection, and change scoping.

| User phrase (any of) | Invoke | Workflow |
|----------------------|--------|----------|
| `release my architecture` · `release architecture` · `release architecture version` · `publish architecture` · `ship architecture` · `tag architecture version` · `freeze architecture` · `bump architecture version` · `finalize architecture` | `sa-skills:architecture-docs` | Workflow 10 — `RELEASE_WORKFLOW.md` (Draft → Released, CHANGELOG, `architecture-v{version}` git tag). PR by default (workflow creates `release/architecture-v{version}` branch, commits, tags, pushes, prints host-agnostic PR instructions). Append `--direct` to commit to the current branch instead. |
| `generate my architecture diagrams` · `create architecture diagrams` · `add diagrams to my architecture` · `update diagrams` | `sa-skills:architecture-docs` | Workflow 8 — diagram generation |
| `migrate my architecture` · `restructure architecture` · `split ARCHITECTURE.md` | `sa-skills:architecture-docs` | Workflow 9 — multi-file `docs/` layout migration |
| `review my architecture` · `peer-review architecture` · `architecture quality review` | `sa-skills:architecture-peer-review` | Full peer review with HTML playground |
| `generate compliance contracts` · `generate compliance documentation` · `compliance contracts` | `sa-skills:architecture-compliance` | 10 compliance contracts (Contratos de Adherencia) |
| `check compliance health` · `compliance portfolio review` · `concept gaps across contracts` | `sa-skills:architecture-compliance-review` | Portfolio gap explorer playground |
| `check requirements coverage` · `traceability report` · `PO spec coverage` · `deviation check` | `sa-skills:architecture-traceability` | `TRACEABILITY_REPORT.md` |
| `generate dev handoff` · `component handoff for <X>` · `implementation handoff` | `sa-skills:architecture-dev-handoff` | Per-component **8-section audience-segmented handoff** (Dev / QA / Ops tracks; TEMPLATE_VERSION 2.0.0) + scaffolded assets. Existing v1 (16-section) handoffs auto-regenerate to v2 silently on the next run. Optional flags: `--parallelism N` (default 4, capped at 8) — Stage 5A handoff-document batch size (sonnet); `--asset-parallelism N` (default 4, capped at 8, **v3.14.7+**) — Stage 5B asset batch size (sonnet for code-style assets, haiku for descriptor-style); `--force` regenerates all components, bypassing the `handoffs/.manifest.json` skip-if-unchanged check. |
| `add component <X>` · `remove component <X>` · `sync component index` · `migrate to C4` | `sa-skills:architecture-component-guardian` | `docs/components/README.md` (only sanctioned writer) and `<!-- EXPLORER_HEADER -->` blocks for every component file it writes (L1 descriptors + L2 containers) |
| `export architecture to Word` · `export handoff to Word` · `export security posture` · `generate .docx` | `sa-skills:architecture-docs-export` | `.docx` export only |
| `run architecture analysis` · `SPOF analysis` · `blast radius analysis` · `STRIDE threat model` · `cost hotspot analysis` · `tech debt analysis` · `security posture` · `security posture validation` · `all analyses` | `sa-skills:architecture-analysis` | Risk & sustainability reports in `analysis/` — output format is asked at runtime (Step 2.4) after analysis selection: **Markdown** (`.md`, grep-friendly, default) OR **HTML** (`.html`, interactive d3.js v7 + d3-sankey, corporate styling, requires internet for viz). The same findings populate either format. |
| `onboard to architecture` · `architecture concept map` · `new team member onboarding` | `sa-skills:architecture-onboarding` | Canvas concept-map playground |
| `PO spec` · `elicit requirements` · `business context intake` · `evaluate PO spec` | `sa-skills:architecture-readiness` | Requirements elicitation + scoring |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shadowX4fox/solutions-architect-skills](https://github.com/shadowX4fox/solutions-architect-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-26 -->
