---
trigger: always_on
description: > **Status: Alpha.** APIs are evolving rapidly. Breaking changes happen without notice.
---

# phase agent instructions

> **Status: Alpha.** APIs are evolving rapidly. Breaking changes happen without notice.

## Repository overview

Animation infrastructure for the web. Lifecycle-aware primitives that compose visibility, timing, reduced motion, and quality signals into coherent state machines with debuggable transitions.

Tech stack: TypeScript, tsdown (bundler), vitest (tests), oxlint (linting), oxfmt (formatting), lefthook (pre-commit hooks).

## Commands

```bash
pnpm build             # Build (tsdown)
pnpm test              # Run tests (vitest)
pnpm typecheck         # Type check (tsc --noEmit)
pnpm lint              # Lint (oxlint)
pnpm lint:fix          # Lint and auto-fix (oxlint --fix)
pnpm format            # Check formatting (oxfmt --check)
pnpm format:fix        # Fix formatting (oxfmt)
pnpm size              # Build and check bundle sizes (size-limit)
pnpm size:readme       # Build and update README.md bundle size table
pnpm validate          # Run typecheck, lint, format, test, and skill:check in parallel
pnpm skill:check       # Verify every public export has a skill reference (drift guard)
pnpm skill:build       # Regenerate skills/phase/metadata.json from SKILL.md
pnpm skill:package     # Rebuild the deterministic skills/phase/dist/phase-skill.zip
```

**When these run automatically:**

- `skill:check` runs as part of `pnpm validate` and in CI on every PR.
- `skill:build` + `skill:package` run on pre-commit (via lefthook) whenever a file under `skills/phase/` is staged, and Lefthook re-stages the regenerated `metadata.json` and zip automatically. You rarely need to run them by hand.
- CI re-verifies all of these on every PR; the release workflow re-verifies them before publishing. A stale generated file fails the build.

## Architecture

Two-layer design: core primitives (framework-agnostic) and React bindings.

### Top-level folders

| Folder   | Purpose                                                                                                                                                                                |
| -------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `ease/`  | Pure math (easing functions, clamping, interpolation). No browser APIs, no React. Safe anywhere.                                                                                       |
| `core/`  | Framework-agnostic primitives (ticker, sight, loop). Browser APIs live here. `_internal/` holds shared infrastructure (error factory, observer pools) that is never exported publicly. |
| `react/` | React hooks and components. Each hook or component gets its own folder. Depends on `core/` for the underlying primitives. `_internal/` holds shared hooks not exported publicly.       |
| `tests/` | Shared test infrastructure (mock factories for IO/RO/MQL, performance regression tests).                                                                                               |

### File organization conventions

- **One folder per module.** Each function, hook, or component lives in a named kebab-case folder with `index.ts` (implementation) and `index.spec.ts` (tests) co-located.
- **Barrel files.** Only three exist: `src/index.ts`, `src/ease/index.ts`, `src/react/index.ts`. These serve as entry points. Don't add more.
- **`_internal/` directories.** Keep `_internal/` helpers within their layer. Don't export them publicly or import them across layers.
- **New modules follow the existing pattern.** Look at any sibling folder. Match the structure.

### Entry points

| Export    | Source               | Contents                                                                               |
| --------- | -------------------- | -------------------------------------------------------------------------------------- |
| `.`       | `src/index.ts`       | createTicker, createSight, createLoop, createScrollProgress, easing, errors            |
| `./ease`  | `src/ease/index.ts`  | Easing + math utilities (clamp, lerp, remap, easeOutCubic)                             |
| `./react` | `src/react/index.ts` | useLoop, useTween, useCanvas, useSight, useScrollProgress, usePresence, Presence, Swap |

## Performance contracts

Every module must satisfy these performance contracts without exception.

### Hot-path rules (per-frame code)

- **Zero allocations per frame.** No object/array/string creation, no closures, no `.map()`, `.filter()`, spread operators, or template literals in the onTick path.
- **FrameState mutated in place.** Sealed shape, V8 stays on monomorphic IC path.
- **No try/catch wrapping onTick.** Defeats TurboFan optimization.
- **Stable function references.** Frame callback created once, never recreated.
- **No debug logging in hot path.** Zero string ops unless devtools active.

### Lifecycle rules

- **Strong pause.** `cancelAnimationFrame()` stops scheduling entirely. Zero callbacks fire, and the loop fully halts (no rAF + early return).
- **Frame-locked shared clock.** One `performance.now()` read per rAF frame. All tickers read from this shared value. No visual desync.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vercel-labs/phase](https://github.com/vercel-labs/phase) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-10 -->
