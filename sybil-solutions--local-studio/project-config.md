---
trigger: always_on
description: Instruction manual for an LLM launched into this repository. Read it before
---

# AGENTS.md

Instruction manual for an LLM launched into this repository. Read it before
touching anything, then obey it for every turn.

## What this is

Local Studio — a local-first workstation for running, managing, and using
self-hosted LLM backends. Three modules share one controller API:

- `controller/` — Bun/Hono backend (model lifecycle, OpenAI-compatible proxy,
  GPU/system state, SSE, downloads, settings).
- `frontend/` — Next.js 16 + React 19 UI and the Electron desktop shell. Hosts
  `/agent` (Pi coding agent runtime), settings, usage, recipes, logs, setup.
- `cli/` — Bun command-line client for a controller.

One machine launches models, watches GPU/runtime state, chats with
OpenAI-compatible endpoints, and runs agent sessions against local or remote
controllers. See README.md for the system outline and architecture diagrams.

## Working agreement

- **Do not ask questions during execution.** Pick the most sensible default,
  proceed, and surface assumptions in the handoff summary. Only stop when
  genuinely blocked (missing credentials, destructive action needing approval).
- Prefer momentum over permission. Make a decision, record it, keep moving.
- Never print, log, commit, or otherwise expose credentials.
- Never bypass git hooks with `--no-verify`.

## Code standards (non-negotiable)

These apply to all new and edited code. Existing code is legacy; do not
"fix" it broadly, but anything you touch must meet these bars.

### No comments

- **Do not leave comments anywhere.** No `//` explanations, no block comments,
  no commented-out code, no `// TODO`. Code is self-documenting.
- If a comment feels necessary, the real fix is to extract a named function so
  the name carries the intent. Do not "leave a note for later."
- Do not add or expand JSDoc unless a public API contract genuinely requires it;
  prefer a precise type signature over prose.

### One function, one thing

- Keep functions clean and composable: one function does one thing.
- If a function does two things, split it into two named functions and compose
  them. Name functions after what they return/produce, not what they do
  incidentally.
- Pure helpers over inline branching. Prefer early returns over nested nests.

### DRY

- Do not duplicate logic. If two call sites share logic, extract it once.
- Do not reinvent types; reuse `shared/contracts` and existing `@/lib` types.
- The `check:dupes` (jscpd) and `check:deadcode` (knip) gates enforce this.

### Fewest lines that say it

- It is always preferable to accomplish the task in as few lines of code as
  possible, without sacrificing the rules above. No clever golfing that hides
  intent; just no redundant scaffolding, no boilerplate that a helper already
  covers, no five-step ceremony where one composed call suffices.

### Effect — the runtime pattern

Use the Effect pattern for async, scheduling, fibers, and streaming. Docs:
<https://effect.website/docs> (Effect core) and
<https://effect.website/docs/schema> (`@effect/schema`). Follow the Effect v4
idioms already established in this repo.

- Prefer `Effect.gen`, `Effect.sync`, `Effect.tryPromise`, `Effect.sleep`,
  `Schedule.spaced`/`Schedule.exponential`, and `Stream` over hand-rolled async.
- Fork interruptible fibers with `Effect.runFork` and clean them up on unmount.
  Real examples: `src/lib/effect-timers.ts` (`effectInterval`/`effectTimeout`),
  `src/features/agent/runtime/effect-coalescer.ts`.
- **`useEffect` is banned** in feature/render code. Use `useSyncExternalStore`
  with a module-level store whose `subscribe` owns Effect fibers (see
  `src/hooks/realtime-status-store.ts`, `src/hooks/use-controller-events.ts`).
  The few surviving `useEffect` calls are error-boundary legacy; do not add more.
- Validate data at boundaries with `@effect/schema`, not ad-hoc guards.

### UI — use the kit, do not reinvent

- Build UI from the primitives in `frontend/src/ui/` (barrel: `@/ui`). Use
  `Button`, `Input`, `Select`, `Textarea`, `Checkbox`, `FormField`,
  `SegmentedControl`, `Tabs`, `Card`, `Alert`, `Table`/`THead`/`TBody`/`TRow`/
  `TH`/`TCell`, `ListGroup`/`ListRow`, `AppPage`/`PageHeader`, `Modal`, `Slider`,
  `ProgressBar`, `Stat`, `ErrorBox`, `MarkdownContent`, etc. Do not hand-roll
  controls, dialogs, or layout shells that the kit already provides.
- Layering is enforced by `scripts/validate-ui-structure.mjs` and the
  `check:ui-structure` gate:
  - `src/ui` — shared primitives only; never import features or app code.
  - `src/features` — one folder per page-feature; never import app code.
  - `src/app` — thin route shells composing features; no `_components` trees.
  - `src/components` — retired; must stay empty.
  - `src/lib`, `src/hooks` — shared layer; every module needs consumers in more
    than one feature (the gate fails otherwise).
- Use the ZCode design tokens in `src/app/styles/globals/tokens.css`
  (`--color-*`, `--surface`, `--border`, `--fg`, `--dim`, `--accent`, `--fs-*`).
  Do not hard-code colors or spacing; reach for a token.

### TypeScript

- No `any`. No `@ts-ignore`. No unchecked casts. If the type is wrong, fix the
  type. `@/` path alias maps to `src/`.
- `typecheck` and `typecheck:desktop` must stay green.

## Validation (run before handoff)

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sybil-solutions/local-studio](https://github.com/sybil-solutions/local-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-27 -->
