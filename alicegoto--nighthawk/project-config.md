---
trigger: always_on
description: Reply in the same language as the user.
---

# Repository-level Agent Guide

Reply in the same language as the user.

This is a TypeScript monorepo built for agent-assisted development. Keep the root `AGENTS.md` limited to hot-path rules: the project map, hard constraints, and workflow requirements — things every task needs to know.

## Project Overview

**NightHawk** is a security-first AI agent for the terminal — penetration testing, code audit, and full-strength coding in one loop. It pairs a modern coding agent core (Plan/Act/Observe/Reflect loop, sub-agents, MCP, skills, persistent memory) with a native security engine — 116+ vulnerability rules mapped to OWASP Top 10 and CWE, Shannon-entropy secret detection, cross-file taint analysis, and dependency auditing (offline, OSV, and host package-manager) — all exposed as first-class tools the agent can invoke mid-session.

### Technology Stack

- **Language**: TypeScript (strict mode, ES2024 target, bundler module resolution)
- **Package Manager**: pnpm 10.33.0 (monorepo workspaces)
- **Runtime**: Node.js ≥ 24.15.0
- **Build Tools**: tsdown (for bundling), TypeScript 6.0.2
- **Testing**: Vitest 4.1.4 (unit/integration tests), custom smoke tests, node:test for pi-tui
- **Linting**: oxlint 1.59.0 with TypeScript-aware rules
- **Formatting**: oxlint auto-fix + lint-staged with git hooks
- **CI/CD**: GitHub Actions (sharded test runs, linting, typechecking, security smoke tests)
- **Release Management**: Changesets for versioning and changelogs
- **Documentation**: VitePress for bilingual docs (English/Chinese)
- **Nix**: Flake-based build and dev shells for reproducible environments

### Key Architecture Components

1. **Agent Core Engine** (`packages/agent-core-v2`): The main agent engine with DI × Scope architecture (App/Workspace/Session/Agent scopes)
2. **KAP Server** (`packages/kap-server`): NightHawk server exposing REST + WebSocket APIs
3. **LLM Abstraction** (`packages/kosong`): Provider-agnostic LLM integration (OpenAI, Anthropic, Google, DeepSeek)
4. **Execution Environment** (`packages/kaos`): File/process abstractions for local/remote execution
5. **Security Engine**: 116+ vulnerability rules, secret scanning, taint analysis (production code in `packages/agent-core/src/tools/builtin/security/`)
6. **Terminal UI** (`packages/pi-tui`): Component framework for the TUI
7. **Client SDK** (`packages/klient`): Contract-driven facade over agent-core-v2

## Working Principles

- Think from first principles. Start from real requirements, code facts, and verification results; if the goal is unclear, discuss it with the user first.
- Treat code, not documentation, as the source of truth. Unless the user explicitly says otherwise, do not read ordinary Markdown just to understand the implementation.
- Before making code changes, read the relevant code and the most recent constraints, and follow the nearest `AGENTS.md` in the directory tree.
- Keep changes focused. Do not slip in unrelated refactors along the way.
- When committing, do not add any co-author attribution, and do not reveal the identity of the agent in commit messages, PR descriptions, or any explanatory text.

## Project Map

### Applications
- `apps/nighthawk`: The CLI / TUI application. It consumes core capabilities through `@nighthawk/nighthawk-sdk` and must not depend directly on `@nighthawk/agent-core`. When writing or modifying its terminal UI, use the `write-tui` skill (`.agents/skills/write-tui/SKILL.md`).
- `apps/vscode`: VS Code extension for NightHawk.
- `apps/nighthawk-inspect`: Web inspector for the kap-server `/api/v1/debug` RPC surface — workspace/session browser, per-session transcript chat, per-scope Service panels, and the DI unit inspection view. See `apps/nighthawk-inspect/AGENTS.md`.
- `apps/vis`, `apps/vis/server`, `apps/vis/web`: Visual debugging tools for sessions and replays.
- the browser web UI: **Its source no longer lives in this repo.** It is developed in the code-app repo (`apps/web`) and shipped as the committed, prebuilt bundle `apps/nighthawk/dist-web` (gitignored, force-added), synced from code-app with `NIGHTHAWK_REPO=<this checkout> pnpm run sync:web` — sync and commit the bundle in the same change whenever the web UI should ship differently. `apps/nighthawk/scripts/check-web-assets.mjs` guards packaging against a missing bundle. To hack on the web UI against this repo's server, run `pnpm dev:server` here and point code-app's `pnpm dev:web` at it via `NIGHTHAWK_SERVER_URL`.

### Core Packages
- `packages/agent-core`: The unified agent engine (v1), including Agent, Session, profile, skills, tools, plan, permission, background, records, the in-process DI service layer (`src/services/`), and other core capabilities. See `packages/agent-core/AGENTS.md`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AliceGoto/nighthawk](https://github.com/AliceGoto/nighthawk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
