---
trigger: always_on
description: Wayflow is an embeddable workflow/graph editor library. Plain TypeScript and the DOM — no UI framework. Everything composes as `createX(params)` returning an `HTMLElement` or a handle object.
---

# Wayflow

Wayflow is an embeddable workflow/graph editor library. Plain TypeScript and the DOM — no UI framework. Everything composes as `createX(params)` returning an `HTMLElement` or a handle object.

## Project structure

A monorepo of layered packages. Each depends only on the ones listed above it, and lower layers never reference upper-layer concepts in names or comments:

- `core` — generic graph primitives (Graph/Node/Edge, viewport, history). No dependencies; no `workflow`/`LLM`/`agent`/`tool` vocabulary.
- `dom` — the DOM canvas/editor (→ core). Stays generic; no `WorkflowEditor` or workflow concepts.
- `agent` — workflow node types, the runtime handler API, validation, `WayflowError` (→ core).
- `runtime` — the execution engine (→ agent, core).
- `models` — provider-neutral LLM/image adapters (→ agent, core, runtime).
- `ui` — the `WorkflowEditor` and design-system primitives (→ core, dom, agent).

## Commands

The repo uses pnpm. Build is [tsdown](https://tsdown.dev) (Rolldown-powered), config in `tsdown.config.ts`.

- `pnpm build` — build all packages to `dist/` (also typechecks via `--dts`). Fast enough that there are no per-package build scripts.
- `npx tsc -b packages/<pkg>` — fast typecheck of a single package.
- `pnpm check` — Biome format + lint check; `pnpm check:write` to autofix; `pnpm format` to format only.
- `pnpm test` — Vitest (`tests/**/*.test.ts`); `pnpm test:watch` for watch mode.
- `pnpm typecheck:tests` — typecheck the `tests/` tree (`tsconfig.test.json`).

A green build is the type check; Biome enforces format/lint. The pure packages — `core`, `agent`, `runtime` — have unit suites under `tests/`; the DOM-bound `dom`/`ui` have none yet, so verify those by running an example app under `examples/` (`quickstart`, `custom-nodes`, `low-level`, `with-backend`). CI (`.github/workflows/ci.yml`) runs format check + lint + build + typecheck-tests + test on every PR.

## Formatting

Biome (`biome.json`): no semicolons, single quotes, 2-space indentation, 80-col width. Run `pnpm format`. When something isn't specified here, follow the conventions already used in the surrounding code.

Relative imports are extensionless (`from './protocol'`, never `'./protocol.js'`) — the build resolves them, and the whole codebase is consistent on this.

## Code style

- Group code into sections with banner comments, with tunable constants at the top of the file. The `–` rule lines run to the 80-col line width: `// ` + 77 en-dashes (`–`, U+2013); for an indented banner, pad so the line still ends at column 80, so a long title never overruns the rule. CSS banners are `/* ` + 74 en-dashes + ` */`.
  ```ts
  // –––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––
  //  Section title
  // –––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––
  ```
- Comment only when the code and names don't already explain it. Never restate the code, narrate a change, or reference a plan; comments must read standalone. Default to none.
- Use common, widely-understood names — never cute ones (`full`/`compact`, not `roomy`/`narrow`).
- Any string-literal value used in more than one place becomes an UPPER_SNAKE `as const` object with a derived type (e.g. `PORT_SIDE`, `TONE`). Never repeat raw enum strings.
- Build views as `createX(params)` factories returning an element or a handle (`{ element, ... }`) — no classes. In large factories, extract each DOM piece into a named `create<Thing>` helper with a typed `Props`; small leaf factories may build inline.
- Prefer `interface`. Avoid casts, and never use `as any`.
- Keep a helper local until a second use, then extract — don't duplicate, and don't abstract on the first use. Prefer reusing or extending an existing primitive over adding a new one.
- Encapsulate DOM-state changes in stateless helpers in the owning module (e.g. `setControlInvalid`); don't leak CSS class names to callers.
- Colors and spacing come from CSS variables that define both light and dark values — never hardcode a theme color.

## Performance

Never tear down and rebuild a DOM subtree on a high-frequency event (stream chunks, run data, `pointermove`, scroll, resize). Build once and update in place: gate renders on whether the displayed data actually changed, and emit what changed so listeners can skip irrelevant updates. For data-driven views, use `UpdatableView<T>` and `createListReconciler` from `packages/ui/src/view.ts`.

## Testing

- Tests live in a separate `tests/<package>/` tree that mirrors the package's `src` layout (no `packages/` prefix); never colocate them with source. The Vitest glob is `tests/**/*.test.ts` only.
- Test through the public surface a consumer actually imports — the published umbrella subpaths (`wayflow/runtime`, `wayflow/core`, …), not the internal workspace names (`@wayflow/*`, which source uses to depend across packages). This catches export-map regressions a `@wayflow/*` import would miss. Black-box: internal helpers (e.g. the scheduler's `applyArrayOp`) are exercised via `run()`, not imported, so they stay free to refactor.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TahaSh/wayflow](https://github.com/TahaSh/wayflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
