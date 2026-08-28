---
trigger: always_on
description: This repository contains **OnPrem AI Gateway** (**OP AI Gateway**, OP = On
---

# AGENTS.md

## Project Summary

This repository contains **OnPrem AI Gateway** (**OP AI Gateway**, OP = On
Premise / On Prem), a Linux-first, cross-platform AI gateway portal, licensed
AGPL-3.0-only.

The system routes AI client requests to AI servers running Ollama, llama.cpp,
or vLLM. It supports OpenAI-compatible clients, Anthropic-compatible clients,
coding agents such as Codex and Claude Code, and a built-in portal chat
experience. It ships with local users, invite-based provisioning, API tokens,
model-based routing with load-aware scoring from agent telemetry, usage
analytics, a multi-platform server-reporting agent, an optional NetBird mesh
with gateway-managed certificates, and a German/English portal UI.

## Documentation Sources Of Truth

Keep each kind of information in exactly one canonical place:

- **`docs/architecture/`** is the canonical description of the system:
  structure, behavior, constraints, decisions, and quality gates. Start at
  [`docs/architecture/README.md`](docs/architecture/README.md) (arc42-style
  index: chapters 01–12, cross-cutting concept docs, and reference docs).
  When behavior or structure changes, the matching architecture document
  MUST be updated in the same unit of work.
- `AGENTS.md` (this file) contains durable repository instructions, required
  workflows, and commands that every coding agent must follow. It points into
  `docs/architecture/` instead of duplicating it.
- `docs/superpowers/specs/` and `docs/superpowers/plans/` are **branch-local
  working documents**: create and use them freely on a feature branch (the
  brainstorming/planning workflow depends on them), but they are **never
  merged to `main`**. Before the branch is finalized for its pull request,
  fold everything durable into `docs/architecture/` and remove the folder
  from the branch. Never cite them as the source of current behavior — the
  architecture docs are.
- `docs/implementation-status.md` is the same kind of **branch-local working
  document**: keep it updated during branch work (handoff state between
  sessions/agents on that branch), fold durable outcomes into
  `docs/architecture/`, and remove it before the pull request is finalized.
  The pull request description carries the final summary; `main` never
  contains this file.
- `CLAUDE.md` is only a compatibility pointer to these files. Never place
  agent instructions, project history, verification logs, or current handoff
  state in it.

When information appears in more than one of these files, remove the
duplicate instead of maintaining parallel copies.

## Branching And Pull Requests

These rules are absolute:

- **Never commit to or merge into `main` yourself.** Every change reaches
  `main` exclusively through a **feature branch and a pull request**; the
  merge is performed by a human reviewer after CI is green. This applies to
  code, docs, and one-line fixes alike.
- Work on a feature branch, preferably in a worktree under
  `.worktrees/<feature-name>`.
- **`docs/superpowers/**` and `docs/implementation-status.md` must not land
  in `main`.** They may — and should — be created and used on the branch
  while working. As the **last step** before the branch is ready for its
  pull request: fold all durable information from them into
  `docs/architecture/`, then delete them from the branch. Verify with
  `git diff --name-only main...HEAD` that neither path appears as added or
  modified in the PR diff.
- Prefer **squash merges** for pull requests so branch-local working files
  never enter `main`'s history.

## Repository Layout

Monorepo. Top-level directories:

- `gateway/backend/`: the Go module `op-ai-gateway` (`cmd/`, `internal/*`).
  Import paths are unprefixed (`op-ai-gateway/internal/gateway`).
- `gateway/frontend/`: the React/TypeScript/Vite portal UI, served under the
  `/portal/` base path.
- `gateway/e2e/`: the Playwright end-to-end suites (own package).
- `gateway/deploy/`: Dockerfiles, `docker-compose.yml`, nginx config, and the
  operator-facing `themes/` directory (Docker build context is `gateway/`).
- `server-agent/`: the standalone server agent (own Go module
  `op-ai-server-agent`, imports nothing from the gateway). Build/test from
  `server-agent/`.
- Repo root: `Makefile`, `scripts/`, `docs/`, `.golangci.yml`,
  `sonar-project.properties`, and the gitignored `data/`, `.worktrees/`,
  `.sonar-local/`.

The authoritative component/package breakdown (both Go modules and the
frontend) is
[`docs/architecture/05-building-block-view.md`](docs/architecture/05-building-block-view.md).
Do not record live branch, worktree, or in-progress-feature state in this
file: inspect Git and `docs/implementation-status.md` at the start of a task.

## Architecture And Constraints — Where To Read

Read the matching document before changing an area. The most load-bearing
ones:

| Area | Canonical document |
|---|---|
| System goals, quality goals | `docs/architecture/01-introduction-and-goals.md` |
| Hard constraints (incl. product naming, licensing) | `docs/architecture/02-constraints.md` |
| External interfaces & context | `docs/architecture/03-context-and-scope.md` |
| Strategy & key mechanisms | `docs/architecture/04-solution-strategy.md` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JLor08/op-ai-gateway](https://github.com/JLor08/op-ai-gateway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
