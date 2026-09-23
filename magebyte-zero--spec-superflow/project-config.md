---
trigger: always_on
description: This file provides guidance to Codex (codex.ai) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (codex.ai) when working with code in this repository.

## Current default (new changes)

Use only when the user opts into spec-superflow. New tasks have direct and planned entry points (`workflow start`), not a five-path questionnaire. Planned work uses proposal.md + tasks.md and one concrete approval; specs/design are optional. No handwritten execution-contract or recommendation receipt is required. Native/final is default, delegation and worktrees are explicit. Ordinary debugging stays in executing. `workflow complete` runs final checks and records verified or explicit accepted-risk delivery; failure is never rewritten as pass. The eight-state and DP descriptions below are legacy compatibility, not additional steps for new changes. User instructions always take precedence.

## What This Is

A self-contained Codex plugin that integrates OpenSpec-style planning + Superpowers execution discipline. Zero runtime dependencies, supports 9 installation surfaces (Claude Code, Cursor, OpenAI Codex CLI, OpenAI Codex App, GitHub Copilot CLI, Gemini CLI, OpenCode, WorkBuddy, Trae).

## Commands

```bash
# Build TypeScript
npm run build

# Run integration tests
npm test

# Run single test (Node 20+ native test runner)
node --test tests/e2e.test.mjs --test-name-pattern="parseDeltaSpec"

# Validate artifacts (uses docs/examples/ data)
npm run validate
```

## Architecture

### Source Code (`src/`)

TypeScript interfaces + regex-based parsers. Compiles to `dist/` (ES2022 + NodeNext + strict).

- `schema/` — Type definitions: `base.ts` (Requirement, Scenario), `change.ts` (Delta operations), `spec.ts`
- `parsing/` — `requirement-blocks.ts` parses delta spec markdown. `change-parser.ts` extracts `## Why` + `## What Changes` + delta sections from proposal markdown.
- `validation/` — `validator.ts` validates artifacts against schema rules. `constants.ts` holds thresholds. All public API re-exported from `src/index.ts`.

### Validation Rules

- **spec.md**: Each Requirement must contain `SHALL` or `MUST`, at least 1 `#### Scenario:` block
- **Delta spec**: ADDED/MODIFIED must have requirement text + scenarios; cross-section conflicts blocked
- **proposal.md**: `## Why` ≥ 50 characters, `## What Changes` cannot be empty
- `Validator` returns `ValidationReport` with `{valid, issues: [{level, path, message}], summary}`. Strict mode treats warnings as errors.

### Skills (`skills/`)

9 skills, one per directory. Each contains a `SKILL.md` that Codex loads as an instruction set:

| Skill | Phase | Purpose |
|-------|-------|---------|
| `workflow-start` | Entry | Content-level state detection, 8-state routing, blocks illegal transitions |
| `need-explorer` | Exploring | One-question-at-a-time elicitation, 2-3 approach comparison with recommendation |
| `spec-writer` | Specifying | Generate planning artifacts + Schema engine validation |
| `contract-builder` | Bridging | Parsing engine auto-extracts 4 planning artifacts → compresses into `execution-contract.md` |
| `build-executor` | Executing | TDD Iron Law + SDD subagent-driven development + Review Gates |
| `bug-investigator` | Debugging | 4-phase root cause analysis. 3+ fix failures → question architecture → escalate |
| `code-reviewer` | Review | Structured review with 3 severity levels (Critical/Important/Minor) |
| `release-archivist` | Closing | Verification-before-completion Iron Law, archiving, risk summary |
| `spec-merger` | Sync | Delta Spec → intelligent merge into main specs, conflict detection |

### Skill Sub-Prompts

- `skills/build-executor/implementer-prompt.md` — Subagent implementation template with TDD evidence + self-review requirements
- `skills/build-executor/task-reviewer-prompt.md` — Dual-verdict review (spec compliance + code quality)
- `skills/code-reviewer/code-reviewer-prompt.md` — Structured code review template with 3 severity levels

### State Machine

8 states: `exploring`, `specifying`, `bridging`, `approved-for-build`, `executing`, `debugging`, `closing`, `abandoned`.

```
exploring → specifying → bridging → approved-for-build → executing → closing
                ↑              ↑             |                 ↑    |
                |              |             v                 |    |
                |              |         debugging ────────────┘    |
                |              |                                    |
                +--------------+------------------------------------+
                (scope change → re-specify)    (contract drift → re-bridge)
```

`workflow-start` is the single entry point. It reads artifact content (not just file existence) to determine current state.

### Hard Constraints

- No `execution-contract.md` or no user approval → implementation is **blocked**
- Requirements change mid-execution → forced rewind to `specifying` or `bridging`
- Bug encountered → must enter `debugging` state; no "just try random fixes"
- Contract scope drift detected (proposal intent lock ≠ contract intent) → re-bridge

### Helper Scripts (`scripts/`)

- `spec-superflow.mjs` — CLI entrypoint for `ssf` / `spec-superflow` commands.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MageByte-Zero/spec-superflow](https://github.com/MageByte-Zero/spec-superflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
