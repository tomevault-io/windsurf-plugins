---
trigger: always_on
description: NVIDIA NemoClaw is an open-source reference stack for running [OpenClaw](https://openclaw.ai) always-on assistants inside [NVIDIA OpenShell](https://github.com/NVIDIA/OpenShell) sandboxes more safely. It provides CLI tooling, a blueprint for sandbox orchestration, and security hardening.
---

# Agent Instructions

## Project Overview

NVIDIA NemoClaw is an open-source reference stack for running [OpenClaw](https://openclaw.ai) always-on assistants inside [NVIDIA OpenShell](https://github.com/NVIDIA/OpenShell) sandboxes more safely. It provides CLI tooling, a blueprint for sandbox orchestration, and security hardening.

**Status:** Alpha (March 2026+). Interfaces may change without notice.

## Agent Skills

This repo ships agent skills under `.agents/skills/`, organized into three audience buckets: `nemoclaw-user-*` (end users), `nemoclaw-maintainer-*` (project maintainers), and `nemoclaw-contributor-*` (codebase contributors). Load the `nemoclaw-skills-guide` skill for a full catalog and quick decision guide mapping tasks to skills.

## Architecture

| Path | Language | Purpose |
|------|----------|---------|
| `bin/` | JavaScript (CJS) | CLI launcher (`nemoclaw.js`) and small compatibility helpers |
| `src/lib/` | TypeScript | Core CLI logic: onboard, credentials, inference, policies, preflight, runner |
| `nemoclaw/` | TypeScript | Plugin project (Commander CLI extension for OpenClaw) |
| `nemoclaw/src/blueprint/` | TypeScript | Runner, snapshot, SSRF validation, state management |
| `nemoclaw/src/commands/` | TypeScript | Slash commands, migration state |
| `nemoclaw/src/onboard/` | TypeScript | Onboarding config |
| `nemoclaw-blueprint/` | YAML | Blueprint definition and network policies |
| `scripts/` | Bash/JS/TS | Install helpers, setup, automation, E2E tooling |
| `test/` | JavaScript (ESM) | Root-level integration tests (Vitest) |
| `test/e2e/` | Bash/JS | End-to-end tests (Brev cloud instances) |
| `docs/` | Markdown (MyST) | User-facing docs (Sphinx) |
| `k8s/` | YAML | Kubernetes deployment manifests |

## Quick Reference

| Task | Command |
|------|---------|
| Install all deps | `npm install && cd nemoclaw && npm install && npm run build && cd .. && cd nemoclaw-blueprint && uv sync && cd ..` |
| Build plugin | `cd nemoclaw && npm run build` |
| Watch mode | `cd nemoclaw && npm run dev` |
| Run all tests | `npm test` |
| Run plugin tests | `cd nemoclaw && npm test` |
| Run all linters | `make check` |
| Run all hooks manually | `npx prek run --all-files` |
| Type-check CLI | `npm run typecheck:cli` |
| Auto-format | `make format` |
| Build docs | `make docs` |
| Serve docs locally | `make docs-live` |

## Key Architecture Decisions

### Dual-Language Stack

- **CLI and plugin**: TypeScript (`src/`, `nemoclaw/src/`) with a small CommonJS launcher in `bin/`; ESM in `test/`
- **Blueprint**: YAML configuration (`nemoclaw-blueprint/`)
- **Docs**: Sphinx/MyST Markdown
- **Tooling scripts**: Bash and Python

The `bin/` directory uses CommonJS intentionally for the launcher and a few compatibility helpers so the CLI still has a stable executable entry point. The main CLI implementation lives in `src/` and compiles to `dist/`. The `nemoclaw/` plugin uses TypeScript and requires compilation.

### Testing Strategy

Tests are organized into three Vitest projects defined in `vitest.config.ts`:

1. **`cli`** — `test/**/*.test.{js,ts}` — integration tests for CLI behavior
2. **`plugin`** — `nemoclaw/src/**/*.test.ts` — unit tests co-located with source
3. **`e2e-brev`** — `test/e2e/brev-e2e.test.js` — cloud E2E (requires `BREV_API_TOKEN`)

When writing tests:

- Root-level tests (`test/`) use ESM imports
- Plugin tests use TypeScript and are co-located with their source files
- Mock external dependencies; don't call real NVIDIA APIs in unit tests
- E2E tests run on ephemeral Brev cloud instances

### Security Model

NemoClaw isolates agents inside OpenShell sandboxes with:

- Network policies (`nemoclaw-blueprint/policies/`) controlling egress
- Credential sanitization to prevent leaks
- SSRF validation (`nemoclaw/src/blueprint/ssrf.ts`)
- Docker capability drops and process limits

Security-sensitive code paths require extra test coverage.

## Code Style and Conventions

### Commit Messages

Conventional Commits required. Enforced by commitlint via prek `commit-msg` hook.

```text
<type>(<scope>): <description>
```

Types: `feat`, `fix`, `docs`, `chore`, `refactor`, `test`, `ci`, `perf`, `merge`

### SPDX Headers

Every source file must include an SPDX license header. The pre-commit hook auto-inserts them:

```javascript
// SPDX-FileCopyrightText: Copyright (c) 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
// SPDX-License-Identifier: Apache-2.0
```

For shell scripts use `#` comments. For Markdown use HTML comments.

### JavaScript

- `bin/` launcher and remaining `scripts/*.js`: **CommonJS** (`require`/`module.exports`), Node.js 22.16+
- `test/`: **ESM** (`import`/`export`)
- ESLint config in `eslint.config.mjs`
- Cyclomatic complexity limit: 20 (ratcheting down to 15)
- Unused vars pattern: prefix with `_`

### TypeScript

- Plugin code in `nemoclaw/src/` with its own ESLint config
- CLI type-checking via `tsconfig.cli.json`
- Plugin type-checking via `nemoclaw/tsconfig.json`

### Shell Scripts

- ShellCheck enforced (`.shellcheckrc` at root)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NVIDIA/NemoClaw](https://github.com/NVIDIA/NemoClaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->
