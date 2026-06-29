---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A Claude Code plugin that integrates OpenSpec (planning engine) + Superpowers (execution discipline) into a unified AI coding workflow. Self-contained, zero external npm dependencies. 7-platform support (Claude Code / Cursor / Codex / OpenCode / Copilot CLI / Gemini CLI / Trae).

## Commands

```bash
# Build TypeScript
npm run build

# Run integration tests
npm test

# Run single test (Node 22+ native test runner)
node --test --experimental-strip-types tests/e2e.test.ts --test-name-pattern="parseDeltaSpec"

# Validate artifacts (uses docs/examples/ data)
npm run validate
```

## Architecture

### Source Code (`src/`)

TypeScript interfaces + regex-based parsers. Compiles to `dist/` (ES2022 + NodeNext + strict).

- `schema/` — Type definitions: `base.ts` (Requirement, Scenario), `change.ts` (Delta operations: ADDED/MODIFIED/REMOVED/RENAMED), `spec.ts`
- `parsing/` — `requirement-blocks.ts` parses delta spec markdown into `DeltaPlan` (added/modified/removed/renamed blocks). `change-parser.ts` extracts `## Why` + `## What Changes` + delta sections from proposal markdown.
- `validation/` — `validator.ts` validates artifacts against schema rules. `constants.ts` holds thresholds (min Why length: 50, max deltas per change: 10, etc.). All public API re-exported from `src/index.ts`.

### Validation Rules (enforced by `src/validation/validator.ts`)

- **spec.md**: Each Requirement must contain `SHALL` or `MUST`, at least 1 `#### Scenario:` block
- **Delta spec**: ADDED/MODIFIED must have requirement text + scenarios; cross-section conflicts blocked (e.g., same requirement in both MODIFIED and REMOVED)
- **proposal.md**: `## Why` ≥ 50 characters, `## What Changes` cannot be empty
- `Validator` returns `ValidationReport` with `{valid, issues: [{level: ERROR|WARNING|INFO, path, message}], summary}`. Strict mode treats warnings as errors.

### Skills (`skills/`)

9 skills, one per directory. Each contains a `SKILL.md` that Claude Code loads as an instruction set:

| Skill | Phase | Purpose |
|-------|-------|---------|
| `workflow-orchestrator` | Entry | Content-level state detection (compares proposal scope vs contract intent lock), routes to correct skill, blocks illegal transitions |
| `spec-explorer` | Exploring | One-question-at-a-time elicitation, 2-3 approach comparison with recommendation |
| `spec-forger` | Specifying | Generate planning artifacts, runs schema validation on each |
| `bridge-contract` | Bridging | Parsing engine auto-extracts 4 planning artifacts → compresses into `execution-contract.md` |
| `execution-governor` | Executing | TDD Iron Law + SDD subagent-driven development + Review Gates |
| `systematic-debugger` | Debugging | 4-phase root cause analysis. 3+ fix failures → question architecture → escalate |
| `code-reviewer` | Review | Structured review with 3 severity levels (Critical/Important/Minor), invoked after each execution batch |
| `closure-archivist` | Closing | Verification-before-completion Iron Law, archiving, risk summary |
| `spec-syncer` | Sync | Delta Spec (ADDED/MODIFIED/REMOVED/RENAMED) → intelligent merge into main specs, conflict detection |

### Skill Sub-Prompts

- `skills/execution-governor/implementer-prompt.md` — Subagent implementation template with TDD evidence + self-review requirements
- `skills/execution-governor/task-reviewer-prompt.md` — Dual-verdict review (spec compliance + code quality)
- `skills/code-reviewer/code-reviewer-prompt.md` — Structured code review template with 3 severity levels

### State Machine

7 states: `exploring → specifying → bridging → approved → executing → closing`, with a `debugging` side-path from `executing`.

```
exploring → specifying → bridging → approved-for-build → executing → closing
                ↑              ↑             |                 ↑    |
                |              |             v                 |    |
                |              |         debugging ────────────┘    |
                |              |                                    |
                +--------------+------------------------------------+
                (scope change → re-specify)    (contract drift → re-bridge)
```

`workflow-orchestrator` is the single entry point. It reads artifact content (not just file existence) to determine current state.

### Hard Constraints

- No `execution-contract.md` or no user approval → implementation is **blocked**
- Requirements change mid-execution → forced rewind to `specifying` or `bridging`
- Bug encountered → must enter `debugging` state; no "just try random fixes"
- Contract scope drift detected (proposal intent lock ≠ contract intent) → re-bridge

### Helper Scripts (`scripts/`)

- `validate-artifacts` — CLI entry: reads a change directory, validates proposal.md + all specs/*/spec.md, prints a report. Used by `npm run validate`.
- `task-brief` — Extracts a single task's markdown from an implementation plan (avoids pasting task text through controller context).
- `review-package` — Generates a review diff (commit list + stat + diff with extended context) from BASE..HEAD.

### Hooks (`hooks/`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MageByte-Zero/spec-superflow](https://github.com/MageByte-Zero/spec-superflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
