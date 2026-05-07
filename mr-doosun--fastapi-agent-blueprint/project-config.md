---
trigger: always_on
description: This file is the canonical source for project-shared AI collaboration rules.
---

# FastAPI Agent Blueprint — Shared Collaboration Rules

This file is the canonical source for project-shared AI collaboration rules.
Tool-specific harness files must reference this document instead of duplicating its contents.

## Tool-Specific Harnesses

- `CLAUDE.md` — Claude-specific hooks, plugins, slash skills, and tool usage guidance
- `.codex/config.toml` — Codex CLI project settings, profiles, feature flags, and MCP configuration
- `.codex/hooks.json` — Codex command-hook configuration
- `.agents/skills/` — repo-local Codex workflow skills
- `docs/ai/shared/` — shared workflow references consumed by both Claude and Codex
- `.mcp.json` — Claude-only MCP server configuration

### Process Governor Reference Documents

Issue #117 introduced a hybrid local process governor. The four documents below, indexed from [ADR 045](docs/history/045-hybrid-harness-target-architecture.md), define how default coding work is routed:

- [`docs/history/045-hybrid-harness-target-architecture.md`](docs/history/045-hybrid-harness-target-architecture.md) — top-level decisions + design-question resolutions
- [`docs/ai/shared/harness-asset-matrix.md`](docs/ai/shared/harness-asset-matrix.md) — living inventory of every harness asset and its bucket (Keep / Replace / Overlay / Drop)
- [`docs/ai/shared/target-operating-model.md`](docs/ai/shared/target-operating-model.md) — the target workflow, exception model, Claude/Codex alignment, and sample-workflow traces
- [`docs/ai/shared/migration-strategy.md`](docs/ai/shared/migration-strategy.md) — phased migration plan, rollback rules, and the asset-move ordering

Status (2026-05-03 — ADR 047): Phase 5 (#124) shipped the **Hybrid Harness v1** milestone (governor *policy* in a single shared package at [`.agents/shared/governor/`](.agents/shared/governor/), with Claude / Codex hook scripts under `.claude/hooks/` and `.codex/hooks/` as thin shims). [ADR 047](docs/history/047-governor-review-provenance-consolidation.md) right-sizes the steady-state surface: cross-tool review provenance moves to the PR description's `## Governor Footer` block (CI-linted), durable governance constraints live in ADR Consequences (`ADR{NNN}-G{N}` slots), and the per-PR `governor-review-log/` archive is frozen as a closed historical record. The hybrid governance model itself — escape-token vocabulary, dual-tool adapters, scope-of-impact-driven cross-tool review — remains permanent (target-operating-model §3 / §7). Future governor changes belong in the shared package, not in per-tool inline copies (`tests/unit/agents_shared/test_governor_boundary.py` enforces this).

## Project Scale

This project is an AI Agent Backend Platform targeting enterprise-grade services with 10+ domains and 5+ team members.
All proposals and designs must consider scalability, maintainability, and team collaboration at this scale.

## Absolute Prohibitions

- No Infrastructure imports from the Domain layer
- No exposing Model objects outside the Repository
- No separate Mapper classes (inline conversion is sufficient)
- No Entity pattern — unified to DTO (background: [ADR 004](docs/history/004-dto-entity-responsibility.md))
- No modifying or deleting shared rule sources without cross-reference verification
  - Shared rule sources: `AGENTS.md`, `docs/ai/shared/`, `.claude/`, `.codex/`, and `.agents/`
  - Before changing them, verify no dependent tool configs or skills reference the changed content

Note: Domain → Interface **schema** imports (Request/Response types) are permitted.
When fields match, Request is passed directly to Service — creating a separate DTO is prohibited per ADR 004.

## Language Policy

The repository is contributor-facing for both internal teammates and external OSS contributors. Shared rule sources, harness configuration, and AI-governance artefacts must be readable by any contributor regardless of language environment.

The driving failure mode is Korean prose leaking into Tier 1 files via AI sessions running in Korean conversational mode. Other-language leaks have not been observed in this repository, so the **machine-enforced scope today is Korean (Hangul) prose**. The intent of the policy is broader — Tier 1 paths should be English-only — but only Korean is currently detected and blocked. Other-language detection (Chinese, Japanese, etc.) and encoded payloads (base64, HTML entities) are tracked as out-of-scope for this PR; if leaks of those forms appear, expand the checker first, then update this policy text to match.

### Tier 1 paths (Korean prose blocked; English encouraged for everything else)

All new prose, comments, docstrings, log strings, and user-facing terminal output under the following paths should be written in English; **Korean prose is blocked by the pre-commit hook**:

- `AGENTS.md`, `CLAUDE.md`, `CONTRIBUTING.md`, `CODE_OF_CONDUCT.md`, `SECURITY.md`
- `docs/ai/shared/**` (including `governor-review-log/**`)
- `docs/history/**` (every ADR and archive entry)
- `.claude/rules/**`, `.claude/hooks/**`, `.claude/skills/**`
- `.codex/rules/**`, `.codex/hooks/**`
- `.agents/**` (skills and the shared governor package)
- `.github/pull_request_template.md` and `.github/workflows/**`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Mr-DooSun/fastapi-agent-blueprint](https://github.com/Mr-DooSun/fastapi-agent-blueprint) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
