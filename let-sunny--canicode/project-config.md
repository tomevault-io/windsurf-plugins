---
trigger: always_on
description: A CLI tool that analyzes Figma design structures to provide development-friendliness and AI-friendliness scores and reports.
---

# CanICode

A CLI tool that analyzes Figma design structures to provide development-friendliness and AI-friendliness scores and reports.

## Core Goal

**Make the Figma file information-complete so `figma-implement-design` produces accurate code with fewer gotchas.**

canicode's role is upstream of code generation: diagnose where design information is missing (`analyze`), elicit the missing answers from the user (`gotcha-survey`), and write those answers back into the Figma design (`canicode-roundtrip`). Once the design re-analyzes clean, the downstream code-generation step runs in Figma's official `figma-implement-design` skill — canicode does not own that step (see ADR-013 for the scope boundary).

The design-tree format used internally by analysis is a curated, CSS-ready representation; ablation experiments use it as a controlled measurement input. The framing **information curation > information abundance** still drives rule design — fewer information gaps in the source design means cleaner downstream code.

See [Experiment Wiki](https://github.com/let-sunny/canicode/wiki) for detailed data and methodology.

## Target Environment

The primary target is **teams with designers** where developers (+AI) implement large Figma pages:
- **Page scale**: 300+ nodes, full screens, not small component sections
- **Component-heavy**: Design systems with reusable components, variants, tokens
- **AI context budget**: Large pages must fit in AI context windows — componentization reduces token count via deduplication
- **Not the target**: Individual developers generating simple UI with AI — they don't need Figma analysis

This means:
- Component-related rule scores (missing-component, etc.) should NOT be lowered based on small fixture calibration
- Token consumption is a first-class metric — designs that waste tokens on repeated structures are penalized
- Calibration fixtures must be large-scale (270+ nodes) — experiments showed small fixtures (50-100 nodes) produce misleading results
- `no-auto-layout` is the single highest-impact rule (score -10) — empirically validated via ablation experiments

## Tech Stack

- **Runtime**: Node.js (>=18)
- **Language**: TypeScript (strict mode)
- **Package Manager**: pnpm
- **Validation**: zod
- **Testing**: vitest
- **CLI**: cac
- **Build**: tsup

## Project Structure

```
src/                          # Node.js runtime (tsup build)
├── core/                     # Shared analysis engine
│   ├── design-tree/          # Design tree generation, stripping, delta mapping
│   ├── engine/               # rule-engine, scoring, loader
│   ├── comparison/           # visual-compare, html-utils
│   ├── utils/                # fixture-helpers
│   ├── rules/                # Rule definitions + config
│   ├── contracts/            # Type definitions + Zod schemas
│   ├── adapters/             # Figma API integrations
│   ├── gotcha/               # Gotcha survey question generation, instance-context resolution
│   ├── roundtrip/            # Apply gotcha answers to Figma (Plugin API helpers, annotation upsert)
│   ├── report-html/          # HTML report generation
│   ├── monitoring/           # Telemetry
│   └── ui-helpers.ts / ui-constants.ts   # UI helpers shared with app/
├── cli/                      # Entrypoint: CLI
├── mcp/                      # Entrypoint: MCP server
├── agents/                   # Internal: Calibration pipeline (deterministic)
└── experiments/              # Independent experiment scripts (API calls, manual run)
    └── ablation/             # Strip experiments, condition experiments

app/                          # Browser runtime
├── shared/                   # Common UI (gauge, issue list, styles, constants)
├── web/                      # Entrypoint: Web App (GitHub Pages)
│   ├── src/                  # Source
│   └── dist/                 # Build output (deployed)
├── figma-plugin/             # Entrypoint: Figma Plugin
│   ├── src/                  # Source
│   └── dist/                 # Build output (gitignored)

.claude/                      # Claude Code harness
├── docs/                     # Internal reference (read when working on related areas)
│   ├── ADR.md                # Architecture Decision Records (detailed version)
│   ├── ARCHITECTURE.md       # Channels, internal commands, file output structure
│   ├── DESIGN-TREE.md        # Design tree format spec, annotations, conversion examples
│   └── CALIBRATION.md        # Score calibration process, ablation experiments
├── skills/                   # Claude Code skills
│   ├── canicode/             #   CLI wrapper skill
│   ├── canicode-gotchas/     #   Standalone gotcha survey
│   └── canicode-roundtrip/   #   Analyze → gotcha → apply to Figma orchestration
├── agents/                   # Subagents invoked standalone via claude -p
│   ├── calibration/          #   converter, gap-analyzer, critic, arbitrator, runner
│   └── develop/              #   planner, implementer, reviewer, fixer
└── commands/                 # Claude Code commands (calibrate, develop, review-run)

scripts/                      # Orchestration + automation scripts

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [let-sunny/canicode](https://github.com/let-sunny/canicode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-25 -->
