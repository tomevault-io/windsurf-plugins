---
trigger: always_on
description: Monorepo for [Beancount.io](https://beancount.io/) — double-entry bookkeeping made easy.
---

# Beancount.io Monorepo

Monorepo for [Beancount.io](https://beancount.io/) — double-entry bookkeeping made easy.

This file holds repo-wide rules. Per-package guidance lives next to the code:

- `cli/CLAUDE.md` — Python CLI and vendored Fava reporting code
- `dashboard/CLAUDE.md` — web client
- `mobile/CLAUDE.md` — React Native app
- `backend-cluster/backend-v2/CLAUDE.md` — API gateway and background services
- `backend-cluster/ledger/CLAUDE.md` — rustledger-WASM ledger service
- `backend-cluster/idl/CLAUDE.md` — OpenAPI contracts and generated clients
- `backend-cluster/agent-box/CLAUDE.md` — Cloudflare Worker control plane for the Ask-AI sandbox (Claude Code in Cloudflare Sandbox)
- `deploy/CLAUDE.md` — local and hosted deployment targets
- `skills/CLAUDE.md` — agent skills package

## Codex and Claude Code compatibility

- `CLAUDE.md` is the canonical instruction file at every scope. The adjacent `AGENTS.md` must be a relative symlink to it so Claude Code and Codex always read the same instructions; never maintain duplicate copies.
- When adding, moving, or removing a scoped `CLAUDE.md`, make the same structural change to its `AGENTS.md` symlink. When editing either name, update the canonical `CLAUDE.md` through the symlink rather than replacing the symlink with a regular file.
- Shared skills live in `skills/.claude/skills/`. The root `.claude/skills` (Claude Code) and `.agents/skills` (Codex) symlinks must both continue to point there — as the relative link `../skills/.claude/skills` — so both agents use the same skill implementation. Never create a real directory at either path. Edit the canonical skill tree only; do not create divergent Claude-only and Codex-only copies.
- Slash commands are skills. Every `/name` workflow lives at `skills/.claude/skills/<name>/SKILL.md` (with `allowed-tools` in its frontmatter when it needs pre-approved tools); there is no `.claude/commands/` at the root. A command there would be invisible to Codex, and Claude Code lets a same-named skill shadow it anyway.
- Write instructions and skills using behavior supported by both Claude Code and Codex. If platform-specific configuration or tooling is unavoidable, label it clearly and provide equivalent behavior for the other agent.
- After changing instruction files, skills, or their symlinks, run `python3 scripts/check-agent-guidance.py`. It verifies every tracked scope, including nested feature guides, plus the shared-skills link.

## Packages

| Path               | Status | Description                                                                                                                                                                                                |
| ------------------ | ------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `dashboard/`       | active | Web client (React 19, TanStack Start, Apollo, TypeScript)                                                                                                                                                  |
| `mobile/`          | active | React Native iOS/Android app (Expo, Apollo, TypeScript)                                                                                                                                                    |
| `cli/`             | active | `beancount-io` — the `bea` command: directives, bean-check/format, BQL queries, reports, local-ledger ask (Python, Typer) — includes vendored `fava` reporting library. Ships to PyPI and the `bex-co/homebrew-tap` Homebrew tap on `cli-v*` tags |
| `backend-cluster/` | active | Backend services: `backend-v2` (GraphQL/REST/MCP API), `ledger` (rustledger-WASM ledger service), `idl` (OpenAPI specs + generated clients), `agent-box` (Cloudflare Worker sandbox control plane)         |
| `skills/`          | active | Agent skills: the `beancount-*` ledger suite (init, import, importer-author, reconcile, migrate, ask, close, options), the `routine-*` codebase-maintenance suite (logic-simplifier, logic-bugfixer, dup-unifier, dead-code-removal, useless-test-pruner, shipped-feature-inliner, flaky-test-fixer, abstraction-improver, abstraction-police), plus mermaid, pm, pm-brainstorm, loop-worker, ship (see `skills/CLAUDE.md`) |
| `deploy/`          | active | Deployment targets: `deploy/docker-mac/` (Docker Compose, full stack locally), `deploy/dev-sandbox/` (full stack + Ask-AI sandbox for development), `deploy/docker/` (single-host production), and `deploy/bex/` (bex PaaS, no persistent disks — Blueprint at root `bex.yaml`) |
| `docs/`            | active | Documentation content; `docs/adrs/` centralizes every package's Architecture Decision Records (`ADR<NNN>-<package>-<slug>.md`)                                                                             |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bex-co/beancount-io](https://github.com/bex-co/beancount-io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
