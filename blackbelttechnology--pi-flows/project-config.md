---
trigger: always_on
description: `@blackbelt-technology/pi-flows` — a **pi-package** that adds multi-agent workflow orchestration to [pi](https://github.com/earendil-works/pi-coding-agent). Flows are YAML DAGs of agent steps, scheduled in parallel, rendered in a live TUI dashboard.
---

# pi-flows

## Project Overview

`@blackbelt-technology/pi-flows` — a **pi-package** that adds multi-agent workflow orchestration to [pi](https://github.com/earendil-works/pi-coding-agent). Flows are YAML DAGs of agent steps, scheduled in parallel, rendered in a live TUI dashboard.

- **Type:** pi-package (`pi.extensions` manifest in `package.json`). No build step — `main` points directly at `./extensions/index.ts`, executed by pi at load time.
- **Runtime peer:** `@earendil-works/pi-coding-agent ^0.74.0` (plus `pi-ai`, `pi-tui`, `@sinclair/typebox`).
- **Companion repo:** [`pi-agent-dashboard`](https://github.com/BlackBeltTechnology/pi-agent-dashboard) — browser-side flow rendering lives there as a workspace plugin. pi-flows emits `flow:*` events; the dashboard reacts.

## STOP — Docs-First Gate

**Two doc trees:**
- `docs/` — human-readable prose. Reference when answering the user or writing prose for humans.
- `agent-docs/` — caveman-style mirror of `docs/`. **This is what agents read for grounded answers.** Same filenames, terser content.

**Before any build / run / install / deploy / authoring / "how do I X" question: `grep -ni <keyword> README.md agent-docs/*.md` FIRST.** Fall back to `docs/*.md` only if the topic isn't mirrored yet. No source reads until both return nothing.

- ❌ User: "how do I write a flow?" → read `extensions/flow-engine/*.ts` → guess
- ✅ User: "how do I write a flow?" → `grep -ni 'fork\|conditional' agent-docs/flows.md` → quote

If grep finds nothing in either tree, then read source.

## Running, Testing, Deploying

| Task | Command | Notes |
|---|---|---|
| Install in pi (from npm) | `pi install npm:pi-flows` | End-user install. |
| Install local clone | `pi install /path/to/pi-flows` | Dev workflow — pi reads `extensions/index.ts` directly, no build. |
| Lint | `npm run lint` | ESLint flat config in `eslint.config.js`. Scope: `extensions/`, `__tests__/`. |
| Typecheck | `npm run typecheck` | `tsc --noEmit` against `tsconfig.json`. Required for CI. |
| Run tests | `npm test` | Vitest, one-shot. Suites in `__tests__/`. |
| Watch tests | `npm run test:watch` | |
| CI | — | `.github/workflows/ci.yml` runs `lint + typecheck + test` on Node 20/22/24 for every push to `develop` and every PR. |
| Publish | Trigger `Release` workflow in GitHub Actions UI with version input, OR push a `v*` tag | `.github/workflows/publish.yml`. Trusted Publishing via OIDC (`--provenance`), gated by `npm-publish` GH environment. Drafts GitHub Release from CHANGELOG section. See `docs/releasing.md` / `agent-docs/releasing.md`. |
| Use flows in a session | `/flows`, `/flows:new`, `/flows:edit`, `/flows:delete`, `/roles`, `Ctrl+A`, `Ctrl+X` | Flow files at `.pi/flows/flows/<name>.yaml` auto-register as `/<name>`. |

There is **no compile / bundle / dist step**. TypeScript runs straight from `extensions/` via pi's loader. Treat `extensions/index.ts` as the entrypoint.

## Repository Layout

| Path | Purpose |
|---|---|
| `extensions/` | TypeScript source. Entrypoint `index.ts`. Subdirs: `flow-engine/`, `flow-dashboard/`, `flow-context/`, `flow-summary/`, `flow-workspace/`, `shared/`. |
| `agents/` | Built-in agents shipped with the package: `flow-architect.md`, `flow-decision.md`, `project-context-reader.md`. |
| `docs/` | Human-readable reference docs. For users + prose answers. |
| `agent-docs/` | Caveman-style mirror of `docs/` for agent consumption. Same filenames. Grep here first. |
| `__tests__/` | Vitest suites. |
| `openspec/` | Spec-change proposals (see OpenSpec conventions below). |
| `research/` | Exploratory notes, not shipped. |

## Documentation Pointers

Grep `agent-docs/<file>.md` first (caveman, for agents). Fall back to `docs/<file>.md` (human-readable) if not yet mirrored. Same filenames in both trees:

- `README.md` — overview, install, quick start, command list.
- `flows.md` — step types (agent, fork, conditional, agent-loop-decision, flow-ref) with syntax.
- `agents.md` — agent frontmatter schema, model tiers, card types.
- `flow-authoring.md` — full format reference for agent + flow files.
- `architecture.md` — DAG execution, agent isolation, sub-extensions.
- `events-api.md` — register custom cards / tools, listen to `flow:*` events.
- `public-api.md` — exported TypeScript types and functions.
- `tools-reference.md` — built-in tools available to agents.
- `skills-and-extensions.md` — skill bundles + extension registration.
- `creating-packages.md` — author a downstream pi-flow package.
- `extending-pi-flows.md` — advanced customization hooks.
- `dashboard-integration.md` — wire protocol between pi-flows and pi-agent-dashboard.
- `releasing.md` — operator runbook for cutting releases via `.github/workflows/publish.yml`.

## Code Instructions

Behavioral guidelines to reduce common LLM coding mistakes. Bias toward caution over speed. For trivial tasks, use judgment.

### 1. Think Before Coding

- State assumptions explicitly. If uncertain, ask via `ask_user`.
- Multiple interpretations → present them, don't pick silently.
- Simpler approach exists → say so. Push back when warranted.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BlackBeltTechnology/pi-flows](https://github.com/BlackBeltTechnology/pi-flows) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
