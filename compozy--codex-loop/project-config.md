---
trigger: always_on
description: `codex-loop` is a local-first Go CLI and Codex plugin that keeps explicitly activated Codex tasks running until they satisfy either a minimum duration or a target number of deliberate rounds.
---

# codex-loop Agent Instructions

## Project Overview

`codex-loop` is a local-first Go CLI and Codex plugin that keeps explicitly activated Codex tasks running until they satisfy either a minimum duration or a target number of deliberate rounds.

The project ships as:

- a Go single-binary CLI at `cmd/codex-loop`;
- internal runtime packages under `internal/`;
- a Codex plugin bundle under `plugins/codex-loop`;
- a local marketplace entry under `.agents/plugins/marketplace.json`.

Core product premise: every feature must remain usable from the CLI and from Codex lifecycle hooks. Features are incomplete if they only work through internal Go calls.

## Most Critical Rules

### Git Commands Restriction

- **Never run destructive git commands without explicit user permission.**
- Forbidden without explicit permission: `git restore`, `git checkout`, `git reset`, `git clean`, `git rm`.
- Do not restore, remove, or rewrite files that are unrelated to your task.
- If `git status` shows unexpected changes, assume they belong to the user or another agent. Read around them and work with them.

### Plan Mode Persistence

- In Plan mode, after the user accepts a plan, always write the accepted plan to `.codex/plans/`.
- If the accepted plan changes later, update or append the corresponding file under `.codex/plans/`.

### Test Integrity

- Tests exist to discover bugs, not to create mock-driven confidence.
- When a test reveals unexpected behavior, fix production code instead of weakening the assertion.
- Mocks and stubs may be used only at I/O boundaries in unit tests. Final validation for meaningful behavior must include real integration, CLI, hook, filesystem, or end-to-end style checks where applicable.

## Memory Ledger

Maintain one Memory Ledger per agent session in `.codex/ledger/<YYYY-MM-DD>-MEMORY-<slug>.md`.

At the start of every assistant turn:

- read your own ledger file if it exists;
- scan other `*-MEMORY-*.md` files in `.codex/ledger/` for cross-agent awareness;
- treat other agents' ledgers as read-only.

Update your own ledger whenever the goal, constraints, decisions, state, progress, or important tool outcomes change. Keep it short, factual, and compaction-safe. Mark uncertainty as `UNCONFIRMED`.

Use this format:

```markdown
Goal (incl. success criteria):
Constraints/Assumptions:
Key decisions:
State:
Done:
Now:
Next:
Open questions (UNCONFIRMED if needed):
Working set (files/ids/commands):
```

In replies, begin with a brief Ledger Snapshot: Goal, Now/Next, and Open Questions.

## Greenfield OSS v1

- Treat this project as the public `codex-loop` implementation.
- Do not add compatibility aliases, migration bridges, fallback schemas, or references to prior project names unless the user explicitly asks.
- Renames must update code, plugin metadata, docs, tests, marketplace entries, and examples in the same change.
- Delete obsolete code instead of preserving unused compatibility paths.

## Critical Engineering Rules

- `make verify` must pass before completing any code, plugin, or behavior task.
- For Go changes, run `go vet ./...` before or as part of the final gate.
- `make lint` is strict `golangci-lint`; zero warnings and zero issues are acceptable.
- Never add dependencies by editing `go.mod` by hand. Use `go get`, then `go mod tidy`.
- Never use web search for local project code. Use `rg`, `rg --files`, `find`, and local file reads.
- Use external docs only for external APIs, libraries, or current platform behavior. For Codex/OpenAI plugin or hook behavior, prefer official OpenAI documentation.
- Never ignore errors with `_` in production code or tests unless a short written justification is next to the discard.
- Do not commit local scratch, QA, or runtime artifacts such as `.tmp/`, ad hoc Codex homes, coverage files, or generated test sandboxes unless the task explicitly requires them.
- Conversation with the user may be in Brazilian Portuguese. Durable artifacts such as code, comments, docs, plans, commit messages, and this file must be in English.

## Skill Dispatch

Activate skills before writing code or durable project instructions. Use the smallest set that covers the task.

| Domain | Required Skills | Conditional Skills |
| --- | --- | --- |
| Go runtime, CLI, hooks, installer | `golang-pro` | `context7`, `find-docs` |
| Concurrency, cancellation, race fixes | `golang-pro` | `architectural-analysis` |
| Go tests | `golang-pro` | `qa-report`, `qa-execution` |
| Codex plugin metadata, lifecycle hooks, OpenAI behavior | `openai-docs` | `context7`, `find-docs` |
| Release or scenario QA | `qa-report`, `qa-execution` | `golang-pro` |
| Architecture audit, dead code, duplication | `architectural-analysis` | `golang-pro` |
| TUI work, if ever introduced | `bubbletea`, `golang-pro` | |
| External documentation lookup | `context7` or `find-docs` | `exa-web-search-free` |

Every Go change requires `golang-pro`, even when the edit is small.

## Golang Pro Requirements

Follow `golang-pro` as the baseline for all Go work:

- design small interfaces first when behavior crosses package boundaries;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [compozy/codex-loop](https://github.com/compozy/codex-loop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
