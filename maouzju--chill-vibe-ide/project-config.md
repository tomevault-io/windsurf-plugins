---
trigger: always_on
description: **Chill Vibe IDE** — a lightweight, local-first AI IDE for parallel vibe coding.
---

# Repo Notes For AI Agents

## Project Overview

**Chill Vibe IDE** — a lightweight, local-first AI IDE for parallel vibe coding.
Board-first layout with multiple workspace columns, each bound to a provider (`codex` or `claude`) CLI.
Ships as an Electron desktop app.

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | React 19, TypeScript, styled-components, Primer React |
| Backend | Express 5, Node 20+ |
| Desktop | Electron 36, electron-builder |
| Validation | Zod |
| Build | Vite (client), tsup (Electron), ESLint |
| Tests | Node native `--test` runner (unit), Playwright (E2E / visual regression) |
| Music | Netease Cloud Music API |

## Directory Map

```
src/              React app — components, state, styles
server/           Express API — chat, git, providers, music, persistence
shared/           Zod schemas, default state, shared types (imported by both client and server)
electron/         Main process, preload, backend bridge
tests/            Unit tests + Playwright specs + visual regression snapshots
docs/             UI principles, design docs
scripts/          PowerShell/Node dev & test harnesses
build/            Electron build assets (icons, etc.)
```

## First Rule

Changes fall into three tiers with different verification requirements:

### Tier 1 — Logic changes (risk-based TDD)
Applies to: bug fixes, behavior changes, state mutations, server handlers, schemas, Electron bridges.

- Default to strict `red → green` for high-risk logic: bug fixes, reducer/state mutations, persistence or restore paths, server handlers, schemas, Electron bridges, and any change that can corrupt saved state or cross process boundaries.
- For those high-risk changes, write or update a focused test **before** changing production code.
- Run that test first and confirm it **fails** so the issue is demonstrated, not assumed.
- Only after a failing test is observed may you implement the fix.
- Low-risk logic changes may use a prove-after path when red-first would add more ceremony than signal: pure refactors with no intended behavior change, small glue code, logging/telemetry wiring, prototypes, or exploratory spikes.
- The prove-after path still requires targeted verification before handoff: add or update the narrowest relevant test after implementation when practical, or explain why runtime/manual verification is the better fit.
- If you skip red-first for a logic change, say why in the handoff and keep the verification narrow and concrete.
- Before ending your response, rerun the relevant tests and confirm they pass.

### Tier 2 — Style / UI changes (visual regression)
Applies to: CSS, layout, theme tokens, component styling, animations.

- No hand-written unit test required — visual regression snapshots are the primary gate.
- If the change touches a theme-sensitive surface, run `pnpm test:theme` and review snapshot diffs.
- If no existing snapshot covers the changed surface, add one to [`tests/theme-check.spec.ts`](./tests/theme-check.spec.ts).
- Before ending your response, confirm snapshots are updated deliberately, not blindly accepted.

### Tier 3 — Config / docs / tooling (exempt)
Applies to: `AGENTS.md`, `CLAUDE.md`, `docs/`, ESLint config, CI scripts, `package.json` metadata.

- No test required. Verify the change is syntactically valid where applicable (e.g. `pnpm test:quality` for lint/type-check config).

### Shared rule across all tiers
- Do not finish with an unverified code change. Every tier has its own verification step — follow it.

## Docs-first Operating Contract

This section governs agents working on **this Chill Vibe IDE repository**. It is not a product feature and must not be implemented by injecting extra prompts into Chill Vibe-launched Codex/Claude runtime sessions unless the user explicitly asks for that.

- Start every non-trivial repo task from the docs, not from source code. Read `AGENTS.md` first, then the most relevant file under `docs/` or `docs/specs/` before editing production code.
- For net-new features, multi-step behavior changes, cross-cutting product work, or any request that asks for a spec / requirements / design / plan, create or update `docs/specs/<slug>/requirements.md`, `design.md`, and `tasks.md` before touching `src/`, `server/`, `shared/`, or `electron/`.
- For behavior, workflow, UX, packaging, persistence, provider-routing, or test-process changes, update the matching docs in the same task. Code and docs should land together.
- Small scoped bug fixes, pure refactors, and docs/config/tooling-only edits may skip a new SPEC, but the handoff must say why skipping SPEC was reasonable and which existing docs/rules were checked.
- If an agent realizes it started from code without the required docs pass, it must stop, read the relevant docs, and correct the process before continuing.
- Do not treat app runtime prompt changes as a substitute for this repository-level agent contract. The source of truth for agent behavior while developing Chill Vibe is this `AGENTS.md` plus the relevant docs/specs.

## SPEC-first Workflow


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [maouzju/chill-vibe-IDE](https://github.com/maouzju/chill-vibe-IDE) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
