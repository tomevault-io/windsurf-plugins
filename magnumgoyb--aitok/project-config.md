---
trigger: always_on
description: [中文](AGENTS.zh-CN.md)
---

# AGENTS

[中文](AGENTS.zh-CN.md)

This file is the execution guide for AI coding agents working in this repository. User-facing progress and handoff updates should default to zh-CN unless the user asks otherwise. Code, command names, test names, upstream docs, and quoted source text stay as-is.

## 1) Project Mission

`aitok` is an offline-first Go CLI for summarizing local token usage from Claude Code, Codex, and Gemini CLI.
It may estimate USD costs from local token counts using an offline price catalog and local user overrides.
Although it is a command-line tool for humans, product and engineering decisions should prioritize reliable invocation by AI agents and automation.

Current codebase shape:

- Go 1.26.3+
- Single-binary CLI: `cmd/aitok`
- Source adapters: `internal/sources`
- Query aggregation: `internal/query`
- Price catalog and cost estimation: `internal/pricing`
- Report output: `internal/report`
- Gemini local telemetry setup: `internal/setup`
- TUI: `internal/tui`
- Harness sensors: `harness`

Iteration notes:

- Versioned iteration handoff notes live in `docs/iteration-notes/` with zh-CN mirrors in `docs/zh-CN/iteration-notes/`.
- Before continuing agent-cost-governance, pricing coverage, budget, doctor, release, or GitHub automation work, read `docs/iteration-notes/2026-05-09-agent-cost-governance.md`.
- Before continuing TUI period display, local thread/session listing, source title extraction, or the PR #13 release follow-up, read `docs/iteration-notes/2026-05-11-tui-period-threads.md`.

Out of scope unless explicitly requested:

- network upload, remote sync, or cloud reporting
- reading, storing, printing, hashing, or fingerprinting raw API keys
- billing reconciliation or claims of exact provider billing
- turning the TUI or future Web dashboard into a persistent background service

## 2) Dev Commands

- Format and static checks: `make check`
- Enable local git hooks: `make setup`
- Full tests: `make test`
- Harness-only tests: `make test-harness`
- Vet: `make vet`
- Build: `make build`
- Full local validation: `make validate`
- PR metadata check: `make validate-pr-body`
- Commit message check: `make commitlint COMMIT_MSG_FILE=<commit-msg-file>`
- Run CLI directly: `go run ./cmd/aitok summary --period today`

## 3) Iteration Self-Constraint Protocol

Before every product or harness iteration, do a short internal review:

1. Requirement classification
- Classify the request as feature, bugfix, refactor, harness/tooling, or analysis-only.
- State the user-visible outcome, target platform, and affected area.
- State the release decision: engineering/process-only optimization does not require a software release; feature and bugfix work must prompt for or continue into the release flow unless the user explicitly defers it.
- If the request is ambiguous, write the safest concrete assumption before editing. Ask only when a wrong assumption would create meaningful data or product risk.

2. Stack fit
- Prefer the Go standard library and existing package boundaries.
- New dependencies need a concrete reason and must explain impact on binary size, offline behavior, and supply-chain risk.
- Local log parsing must stay streaming; do not load large JSONL logs fully into memory.

3. Acceptance and regression plan
- Write observable acceptance criteria before implementation.
- Cover at least one failure or edge case: empty directories, malformed JSONL, missing fields, duplicate events, Gemini telemetry not configured, or unknown provider.
- Map each criterion to unit tests, harness tests, CLI smoke, manual evidence, or explicit not-applicable rationale.

4. Scope guard
- Name the files or directories expected to change.
- Avoid unrelated refactors, formatting churn, dependency upgrades, or release workflow changes.
- Keep harness, CI, docs, and production code responsibilities separated.

5. Handoff readiness
- Run the smallest targeted check while iterating.
- Run the validation matrix in section 6 before handoff based on changed area.
- Summarize residual risks and any validation that could not be run.

## 4) Coding Rules

- Every source adapter emits `usage.UsageEvent`.
- Do not read, store, or display raw API keys. Provider grouping may only use provider/auth_type metadata already present in CLI logs, or `unknown`.
- Do not add usage-data network transmission. The only command-start network behavior is the low-frequency GitHub release metadata version check, which must never read local logs or upload usage data and must remain skippable with `--no-version-check` or `AITOK_NO_VERSION_CHECK=1`.
- Cost estimation must remain offline by default. Default prices may be updated from public provider pricing, but automatic network sync must be explicitly requested and opt-in.
- Gemini CLI historical data depends on an existing local telemetry outfile. When it is not configured, report no parseable historical data.
- CLI output must stay stable; JSON field changes need tests.
- AI agents should treat `--format json` plus `--no-version-check` as the primary automation contract.
- For JSON commands, stdout must remain a complete machine-readable JSON payload. Human-readable warnings, budget failures, and version prompts belong on stderr or in the returned error path.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MagnumGoYB/aitok](https://github.com/MagnumGoYB/aitok) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-12 -->
