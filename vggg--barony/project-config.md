---
trigger: always_on
description: > This file is contributor/agent-facing. The public story — what Barony is and why —
---

# Barony — Repo Guide

> This file is contributor/agent-facing. The public story — what Barony is and why —
> is `README.md` (outsider front door), with the long form in `docs/concepts.md` and
> `docs/history.md`.

## What this repo is

**Barony** (repo `vggg/barony`; formerly `agent-project-bootstrap`, renamed per ADR-005) — git-native governance for teams of AI coding agents. The **canonical home** for the `barony` skill, the `baron` CLI, and the sister skill `multi-agent-audit` — the runtime-agnostic spec, adapters, references, tests, and meta-docs all live and evolve here.

Current state: **v1.9.0 (pending release)** — one front door (`SKILL.md` routes everything to `skills/barony/assets/collab-repo/START.md`); the legacy v0.3 emit path is quarantined in `legacy/` (deprecated, unmaintained); July-2026 ways-of-working folded in per ADR-002; the `baron` CLI (`cli/`, ADR-003/004) mechanizes the conventions — including the deterministic scaffold `baron init` (ADR-006, templates vendored as package data with a CI drift guard); four runtime adapters (claude, code-puppy, pydantic-ai, generic) with the enforcement-rules artifact (`capability-rules.v1.yaml`) as the single policy source. Track `STATUS.md` for current progress and deferred candidates.

## Canonicality

This repo is canonical for everything. (For the v0→v1 migration story — how canonicality moved here from the vault — see ADR-001 and `CHANGELOG.md`.)

| Surface | Canonical home |
|---|---|
| ADRs (`docs/adr/`) | this repo |
| Runtime adapters (`skills/barony/assets/collab-repo/adapters/{claude,code-puppy,pydantic-ai,generic}/`) | this repo |
| Canonical spec (`skills/barony/references/`) | this repo |
| Acceptance tests (`tests/`) | this repo |
| Emit-time templates (everything else under `skills/barony/assets/`) | this repo |
| Meta-docs (`README.md`, `CLAUDE.md`, `CHANGELOG.md`, `CONTRIBUTING.md`, `STATUS.md`) | this repo |
| `.claude-plugin/plugin.json` | this repo (bumped at release) |

The vault retains a historical copy under `_meta/skills/agent-project-bootstrap/` (the pre-rename skill name); treat it as archival reference, not a source of truth.

## Persona / role for a fresh agent

A fresh Claude Code (or code-puppy, or any other) session landing in this repo operates as a **generic dev archetype**. This repo does not yet dogfood its own multi-persona pattern — there is no `CONVENTIONS.md` / `COORDINATION.md` / `agents/` at the *repo root*. The repo owner works directly with the agent as a single dev. Persona-routing labels (`agent-<name>`) are not defined for this repo.

**Your work queue is [`AGENT-TASKS.md`](AGENT-TASKS.md)** — a prioritized, chase-top-down list (P1 promote-pilot-hardening → P2 new capabilities → P3 dogfood). `STATUS.md` is the canonical progress tracker; `AGENT-TASKS.md` is the ordered queue. When you ship an item, update STATUS.md and propagate the milestone to the vault (see below).

The `CONVENTIONS.md` and `COORDINATION.md` files you'll see under `skills/barony/assets/collab-repo/` are **emit-time templates** that get copied into projects scaffolded BY this skill — they are not this repo's own convention files.

## Propagate project-level updates to the Iris / Brain vault

This repo's *why*-record is local (`STATUS.md`, `docs/adr/`, `CHANGELOG.md`). But **project-level
decisions, milestones, and direction changes must also reach the owner's vault** — Irisidian / Brain at
`/Users/vikram/Obsidian/Brain` — so the vault librarian (**Iris**) can reconcile them into the
cross-project wiki, log, roadmap, and the `AgentBootstrapNasikoMix` project area. There is no automatic
sync. You drop a handoff.

**Propagate ONLY project-level items** (not every commit):
- a new **ADR** or a material **decision**;
- a **release** / version bump / PyPI publish;
- a **direction / roadmap change**, a **phase completion**, a **milestone**;
- a **finding** that changes the product thesis or the pitch.
Do **not** propagate routine commits, refactors, WIP, or internal-only mechanics.

**How** — write a handoff into the vault, then commit + push it there:
- Needs Vikram's input/awareness (decision, direction) → `/Users/vikram/Obsidian/Brain/_handoff/decisions/YYYY-MM-DD-barony-<topic>.md`
- FYI milestone / release / completion → `/Users/vikram/Obsidian/Brain/_handoff/tasks/YYYY-MM-DD-HHMM-barony-<slug>.md`
- Frontmatter (vault schema — `_meta/CONVENTIONS.md § Handoff protocol`):
  ```yaml
  ---
  created: YYYY-MM-DD
  from: Barony
  for: Iris
  status: open
  priority: low | medium | high
  ---
  ```
  Decision notes add `decision: <one-line>` + `urgency:`. Task notes add `task-status: complete | partial | blocked`.
- **Body:** what changed, why it matters at the project level, links (commit SHA / ADR# / PR#), and any owner action needed. Only what Iris needs to reconcile — not the full internal detail; the detail stays in this repo.
- Commit to the **vault repo** with prefix `barony: handoff | …` and push. Iris reads it at her next session, ingests it into `wiki/`+`log`+roadmap, and marks it `done` (never deleted).

If Barony later grows its **own** collab repo + librarian (dogfooding its own multi-persona pattern),

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vggg/barony](https://github.com/vggg/barony) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
