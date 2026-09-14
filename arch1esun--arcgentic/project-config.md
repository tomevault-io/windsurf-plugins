---
trigger: always_on
description: > **This file is auto-loaded by Claude Code at session start.** Read fully before any action. This is your single source of orientation for executing arcgentic v0.2.0.
---

# Arcgentic v0.2.0 — Dev Session Handoff

> **This file is auto-loaded by Claude Code at session start.** Read fully before any action. This is your single source of orientation for executing arcgentic v0.2.0.

---

## 1. Identity

You are a **dev session** continuing arcgentic development.

- **Previous milestone:** v0.1.0-alpha.2 MVP, tagged @ `2efea61` on `main`, fully pushed to `origin/main`. See `CLAUDE-v0.1.0-handoff.md` for that session's context (archived).
- **Current milestone:** v0.2.0 — full role coverage (the P0 group in spec § 19.3 three-phase split).
- **Build contract for this session:** [`docs/plans/2026-05-13-arcgentic-v0.2.0-spec.md`](docs/plans/2026-05-13-arcgentic-v0.2.0-spec.md) (3366 LOC / 23 sections, self-contained — read in full before Task 1).
- **⚠ Spec Amendment 01 — AUTHORITATIVE** (overrides spec § 18 / § 21.5 / § 22 directory paths): [`docs/plans/2026-05-13-arcgentic-v0.2.0-spec-amendment-01-layout.md`](docs/plans/2026-05-13-arcgentic-v0.2.0-spec-amendment-01-layout.md). **Read this BEFORE spec § 18.** Founder ratified the cascade-spec-bug fix on 2026-05-13: hybrid monorepo — markdown skills/agents at repo root (v0.1 plugin contract preserved); Python toolkit at `toolkit/src/arcgentic/` (PyPI-publishable CLI); markdown skills shell out to `arcgentic` CLI.

You execute the spec end-to-end for the **P0 release only** (v0.2.0 release; ~18h estimated). P1 (v0.2.1) and P2 (v0.2.2) are out of scope for this session.

---

## 2. NOT-this (boundary)

| Boundary | Why it matters |
|---|---|
| **NOT Moirai** (`~/Desktop/Arc Studio/Moirai/`) | arcgentic is a separate plugin. Do not touch Moirai files. Do not commit to Moirai repo. (Spec § 23 cites Moirai as origin — that does NOT grant write access.) |
| **NOT v0.1.0-alpha.2 rebuild** | v0.1.0 is closed. Tag `v0.1.0-alpha.2` @ `2efea61` is published. Do not rewrite v0.1.0 history; do not amend any commit prior to HEAD-of-this-session. |
| **NOT v0.2.1 (P1) work** | Per spec § 19.3: P1 = `codify-lesson` skill + `track-refs` skill + `lesson-codifier` + `ref-tracker` agents. These ship in v0.2.1, NOT v0.2.0. |
| **NOT v0.2.2 (P2) work** | Per spec § 19.3: P2 = `cross-session-handoff` skill. Ships in v0.2.2. |
| **NOT a planner role** | Spec is already a plan-level document (more detailed than v0.1.0's plan). Your job is execution, not plan-writing. If you find a spec gap → surface to founder; do NOT silently expand. |
| **NOT an auditor role** | External audit happens after v0.2.0 ships. This session executes only. |
| **NOT IDE adapter beyond spec § 3** | Spec § 3 defines `IDEAdapter` Protocol + 4 implementations. Do not invent new adapters for Cursor variants / JetBrains / etc. — that's v0.3+ scope. |

---

## 3. Where you are (state as of handoff)

| Item | Value |
|---|---|
| Repo local | `/Users/archiesun/Desktop/Arc Studio/arcgentic/` |
| Repo remote | `git@github.com:Arch1eSUN/Arcgentic.git` (PUBLIC) |
| Branch | `main` (tracking `origin/main`) |
| HEAD at handoff | `4813e4e` (sub-phase d.3 + ruff fix; v0.2.0-alpha.1 release happening now) |
| Most recent tag | `v0.1.0-alpha.2` @ `2efea61` (annotated, pushed) |
| Total commits in repo | ~62 (38 pre-MVP/MVP + ~24 v0.2.0 P0) |
| Test files | 9 (`scripts/**/*.test.sh` + `tests/integration/full-lifecycle.test.sh`) — 48 assertions, 100% PASS as of v0.1.0-alpha.2 close |
| Plan file (this session) | [`docs/plans/2026-05-13-arcgentic-v0.2.0-spec.md`](docs/plans/2026-05-13-arcgentic-v0.2.0-spec.md) (3366 LOC) |
| Prior plan (reference) | [`docs/plans/2026-05-12-arcgentic-mvp-plan.md`](docs/plans/2026-05-12-arcgentic-mvp-plan.md) (4580 LOC, COMPLETED) — read for historical patterns; do NOT re-execute |

### v0.1.0-alpha.2 inventory (already in repo — do NOT recreate)

```
plugin.json (version "0.1.0-alpha.2"; status reflects MVP completion)
README.md (updated for alpha.2 + dogfood Gate 1/2 PASS narrative)
README.zh-CN.md (still on alpha.1 — TODO: sync if you touch README in v0.2.0)
LICENSE
.gitignore (includes .claude/ + .agentic-rounds/ + references/ + tracked-skill-references negations)
schema/state.schema.json (minLength on current_round.id removed per Phase 1 plan-bug fix)
scripts/test-helpers.sh
scripts/lib/{yaml.sh, state.sh}
scripts/state/{init.sh, validate-schema.sh, transition.sh, pickup.sh}.sh + .test.sh
scripts/gates/{handoff-doc-gate.sh, round-commit-chain-gate.sh, verdict-fact-table-gate.sh}.sh + .test.sh
tests/integration/full-lifecycle.test.sh
tests/dogfood/replay-r10-l3-llm/RESULT.md
tests/dogfood/gate-2-live-run/RESULT.md
tests/dogfood/gate-3-cross-project/PROTOCOL.md
docs/examples/state.example.yaml
docs/plans/v0.1.0-alpha.2-meta-handoff.md (Gate 2 dogfood artifact)
docs/audits/v0.1.0-alpha.2-meta-external-audit-verdict.md (Gate 2 dogfood artifact)
docs/plans/2026-05-12-arcgentic-mvp-plan.md (MVP plan, COMPLETED)
docs/plans/2026-05-13-arcgentic-v0.2.0-spec.md (this session's build contract)
skills/using-arcgentic/SKILL.md
skills/pre-round-scan/SKILL.md + references/scan-checklist.md
skills/verify-gates/SKILL.md + references/gate-script-catalog.md

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Arch1eSUN/Arcgentic](https://github.com/Arch1eSUN/Arcgentic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-13 -->
