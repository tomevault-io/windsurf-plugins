---
trigger: always_on
description: summary: root routing contract for coding agents working in this repository
---

<!-- docmeta
role: agent
layer: 1
parent: null
children:
  - docs/AGENTS.md
  - src/AGENTS.md
  - scripts/AGENTS.md
  - skills/AGENTS.md
summary: root routing contract for coding agents working in this repository
read_when:
  - first agent entry into the repository
  - need to choose the docs, code, scripts, or skills domain
skip_when:
  - already inside the correct domain router
source_of_truth:
  - AGENTS.md
  - docs/AGENTS.md
  - src/AGENTS.md
  - scripts/AGENTS.md
  - skills/AGENTS.md
-->

# AGENTS.md

Agent router for `telegram-codex-bridge`.

Current shipped truth in this repo is still **Telegram-first**.
Future repository direction may describe a broader **Codex Bridge Core** with additional platform packs.
Keep those two truths separated.

Goal: let a coding agent build a useful mental index of the repo **without preloading the repo**.
The default path is:

1. read this file
2. read **one** domain agent
3. read **one** leaf doc or **one** narrow code file

That is the whole default retrieval path.

## Retrieval Contract

Maximum default depth: **3 layers**.

- Layer 1: `AGENTS.md` (choose the domain)
- Layer 2: one domain agent such as `docs/AGENTS.md` or `src/AGENTS.md`
- Layer 3: one leaf doc or one narrow source file

Do **not** read sibling docs or sibling code files "just in case".
Expand only when the active task is cross-cutting, conflicting, or explicitly architectural.

## Source Priority

Use sources in this order:

1. active user instruction
2. current docs in `docs/product/`, `docs/architecture/`, `docs/operations/`, and `docs/generated/current-snapshot.md`
3. current implementation in `src/`
4. protocol evidence in `docs/research/`
5. planning and history in `docs/roadmap/`, `docs/future/`, `docs/plans/`, and `docs/archive/`

Do not treat planning/history as shipped behavior.
Do not treat protocol capability as proof that Telegram UX already exposes it.

## Domain Router

| If the task is mainly about | Read next |
|---|---|
| current docs, intended behavior, product/runtime/ops questions | `docs/AGENTS.md` |
| current implementation, code ownership, refactors, bug fixes | `src/AGENTS.md` |
| GitHub install entry scripts | `scripts/AGENTS.md` |
| bundled Codex skills | `skills/AGENTS.md` |

## Fast Task Router

| Task | Domain agent | Typical leaf |
|---|---|---|
| v1 boundary, trust model, scope | `docs/AGENTS.md` | `docs/product/v1-scope.md` |
| unsure which Telegram UX doc to use | `docs/AGENTS.md` | `docs/product/chat-and-project-flow.md` |
| auth, project picker, browse, sessions, archive, rename, pin | `docs/AGENTS.md` | `docs/product/auth-and-project-flow.md` |
| Codex-backed Telegram commands, rich inputs, `/review`, `/rollback`, `/compact`, `/model`, `/skills`, `/plugins`, `/apps`, `/mcp`, `/account` | `docs/AGENTS.md` | `docs/product/codex-command-reference.md` |
| runtime cards, `/where`, `/inspect`, `/interrupt`, `/status`, final-answer delivery | `docs/AGENTS.md` | `docs/product/runtime-and-delivery.md` |
| callback payload families and stale/duplicate callback handling | `docs/AGENTS.md` | `docs/product/callback-contract.md` |
| current bridge-versus-platform decoupling progress, what has landed, or what is still Telegram-shaped | `docs/AGENTS.md` | `docs/architecture/platform-decoupling-status.md` |
| runtime lifecycle, SQLite state, recovery, degraded behavior | `docs/AGENTS.md` | `docs/architecture/runtime-and-state.md` |
| current Codex app-server adoption, supported request families, approvals, notification reduction, or rejected server requests | `docs/AGENTS.md` | `docs/architecture/codex-app-server-adoption.md` |
| current pack boundary, active pack selection, platform capabilities, or Telegram/Feishu pack ownership | `docs/AGENTS.md` | `docs/architecture/platform-pack-boundary.md` |
| current Telegram/Feishu capability matrix or future Web/App target capability rows | `docs/AGENTS.md` | `docs/architecture/platform-capability-matrix.md` |
| where ownership lives in `src/` today | `docs/AGENTS.md` or `src/AGENTS.md` | `docs/architecture/current-code-organization.md` |
| current internal Core seam, semantic view contracts, or workflow adapters | `src/AGENTS.md` | one narrow file under `src/core/` |
| new Codex Console continuation task or context-budget decision | `docs/AGENTS.md` | `docs/roadmap/codex-console-continuation-brief.md` |
| future multi-platform Core direction, platform packs, Web/App control surface goals | `docs/AGENTS.md` | `docs/future/multi-platform-core-prd.md` |
| install, config, service, update, diagnostics | `docs/AGENTS.md` | `docs/operations/install-and-admin.md` |
| volatile versions, counts, current size snapshot | `docs/AGENTS.md` | `docs/generated/current-snapshot.md` |
| command registry or Telegram UI implementation | `src/AGENTS.md` | `src/telegram/commands.ts` or one narrow `src/telegram/ui-*.ts` file |
| project/session orchestration implementation | `src/AGENTS.md` | `src/service/session-project-coordinator.ts` |
| Codex command orchestration implementation | `src/AGENTS.md` | `src/service/codex-command-coordinator.ts` |
| runtime-surface implementation | `src/AGENTS.md` | `src/service/runtime-surface-controller.ts` |
| app-server boundary or protocol adoption | `src/AGENTS.md` | `src/codex/app-server.ts` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [InDreamer/telegram-codex-bridge](https://github.com/InDreamer/telegram-codex-bridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-20 -->
