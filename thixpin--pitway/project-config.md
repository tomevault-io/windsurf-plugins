---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Status: Bootstrap Phase

PitWay is being built here, and **PitWay is developed using its own workflow** (dogfooding). The authoritative sources are the workflow state in `.pitway/` (milestones, contracts, backlog) — the original design record is archived at `docs/archive/IMPLEMENTATION_PLAN-v1.0.md` (frozen; never updated).

- Milestones M001–M003 follow the **manual PitWay protocol**: milestone/contract/task artifacts are hand-authored in `.pitway/` following schema v1, commits carry `PitWay-Milestone:` / `PitWay-Task:` trailers, and TDD + atomic-commit rules apply from the first commit. From M004 onward, PitWay manages its own development.
- **Never write implementation code outside a confirmed milestone task.** A milestone's contract must be presented to the developer and explicitly approved in conversation before its baseline commit or any task work.
- Check `.pitway/state.yaml` and the active milestone's `contract.md` / `tasks.yaml` to find current state and next work before doing anything.

There is no build tooling until M001's scaffold task lands. Once `package.json` exists: TypeScript strict / Node ≥ 20 / ESM, `vitest` for tests, runtime deps limited to `commander` + `yaml` + `zod`, MIT license. Add the real commands here at that point.

## What PitWay Is

An npm-distributed CLI (`npx pitway init`; unscoped package name `pitway`) that controls the engineering *process* around AI coding agents — it is not itself an agent. Core philosophy: **"Claude drives the interaction; PitWay controls the workflow state and engineering boundaries."**

Pipeline: Requirement → Milestone → Contract → human confirmation → verification plan → right-sized task graph → task-level TDD (RED→GREEN→REFACTOR) → verification → atomic commit → milestone contract verification → completion.

Non-goals: not a multi-agent framework, coding agent, PM SaaS, CI/CD platform, database-backed orchestration service, or web dashboard.

## Architecture Constraints (binding on all code)

- **Layering**: `CLI → Core → State Store (.pitway/) + Git module`. Claude Code is the primary driver via slash commands, but **the driver never touches `.pitway/` directly** — every state read and mutation goes through a CLI command, and Core validates every transition. This is the load-bearing rule that keeps Core agent-agnostic.
- **Core must never import AI-provider code.** Claude-specific material is text assets only (`src/integrations/claude/`), installed by `pitway init` (default on, `--no-claude` opt-out). Additional drivers may ship as text-asset integrations only, under src/integrations/<driver>/, sharing common assets from src/integrations/common/ via a static, hardcoded common-to-override lookup — never a runtime-loaded plugin system, never dynamic driver registration, never a Core code change, never AI-provider code inside Core.
- **State**: plain files in repo-local `.pitway/` (committed to git), schema v1 with `schema_version: 1` in every file. `contract.md` frontmatter is machine-authoritative (ACs, verification checks, status); the markdown body holds prose and the append-only Change Log. Verification *results* live separately in `verification-results.yaml`. One authoritative source per fact; no derived git data (SHAs) persisted — resolve from commit trailers.
- **Naming**: the racing metaphor (🏁 📜 🛠 🏎 🧪 🔧) is presentation-only, allowed in CLI output. All identifiers, schemas, files, and state values use engineering vocabulary — `status: in_progress`, never `status: racing`.
- **Task states**: `planned → waiting → ready → in_progress → review → completed`, plus `in_progress → blocked → ready`, `in_progress → failed → ready`, `planned|waiting|ready → cancelled`. Milestone states: `draft → confirmed → in_progress → review → completed`, `review → in_progress` on failed verification, `draft|confirmed → cancelled`. No micro-states.

## Key Operational Rules

- **Contract is the execution boundary**: never silently expand scope. A discovered conflict stops work, proposes a contract change (append-only Change Log entry), and waits for developer approval (`milestone-confirm --amend` re-approves).
- **Git safety**: clean working tree required at task *start* (stop and ask on dirty — never auto-stash/reset/absorb). One atomic commit per verified task containing code + same-task state update, with `PitWay-Milestone:` / `PitWay-Task:` trailers; message follows repo convention. Commit messages carry **only** PitWay traceability trailers — never agent/provider session metadata (`Claude-Session:`, `Co-Authored-By: Claude`, `Codex-Session:`, etc.); git history stays provider-agnostic, and workflow state or resume behavior must never depend on an AI conversation or session identifier. Baseline commit at milestone confirmation. Never commit RED states, retries, or intermediate edits. No branches/worktrees/stashes/merges; sequential execution in MVP.
- **Human gate**: `milestone-confirm` runs only after explicit developer approval in the conversation.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thixpin/pitway](https://github.com/thixpin/pitway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-24 -->
