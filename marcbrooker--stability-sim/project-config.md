---
trigger: always_on
description: Stability Sim is a browser-based discrete-event simulation builder for distributed systems. It uses TypeScript, React 19, React Flow, Recharts, Zustand, and Vitest. The simulation engine runs in a Web Worker.
---

# AGENTS.md — Coding Agent Guidelines for Stability Sim

## Project Overview

Stability Sim is a browser-based discrete-event simulation builder for distributed systems. It uses TypeScript, React 19, React Flow, Recharts, Zustand, and Vitest. The simulation engine runs in a Web Worker.

## Build & Test Commands

```bash
npm run build        # Type-check (tsc) then Vite production build
npm test             # Run all tests once (vitest --run)
npm run lint         # ESLint
npm run dev          # Dev server (do not run in automated pipelines)
```

## Code Quality Tasks

### Before Every Change

1. Run `npm run build` to confirm the project compiles cleanly. Fix any type errors before proceeding.
2. Run `npm test` to confirm all existing tests pass.
3. Run `npm run lint` and resolve any warnings or errors.

### When Modifying Simulation Engine Code (`src/engine/`)

- Every component model (`src/engine/components/*.ts`) has a corresponding `.test.ts` file. If you change a component's behavior, update or add tests in the matching test file.
- Property-based tests (files ending in `.property.test.ts`) use `fast-check`. Keep property tests focused on invariants (e.g., priority queue ordering, PRNG determinism, serialization round-trips, metric monotonicity). Do not delete or weaken property tests without justification.
- The simulation engine must remain deterministic for a given seed. Never introduce `Math.random()` or `Date.now()` into engine code — use the `SeededRNG` from `src/engine/prng.ts`.
- The engine runs in a Web Worker (`src/engine/worker.ts`). Do not import DOM APIs or React in any file under `src/engine/`.

### When Modifying Types (`src/types/`)

- All shared types are re-exported through `src/types/index.ts`. If you add a new type file, export it from the barrel.
- Changing a type that appears in the worker protocol (`src/types/worker-protocol.ts`) or serialization models (`src/types/models.ts`) is a breaking change — update the serializers, worker, and all consumers.

### When Modifying UI Components (`src/components/`)

- Custom React Flow node types live in `src/components/nodes/` and are registered in `src/components/nodes/index.ts`. Adding a new component type requires a new node renderer and an entry in that index.
- State lives in Zustand stores (`src/stores/`). Components should read state via selectors, not by importing the store and calling `getState()` in render paths.

### When Modifying Persistence (`src/persistence/`)

- Serializers must maintain the round-trip property: `parse(serialize(x))` must produce a value equivalent to `x`. This is enforced by property-based tests.
- The URL codec (`url-codec.ts`) compresses scenarios with deflate-raw + base64url for `?s=` sharing. It uses the native `CompressionStream` API — no external dependencies.
- Schema migrations (`migrate.ts`) run before validation on every load (JSON file or shared URL). See "When Changing the Saved Scenario Schema" below.

### When Changing the Saved Scenario Schema

Saved scenarios (JSON files and `?s=` URLs) carry a top-level `schemaVersion`. The migration pipeline in `src/persistence/migrate.ts` upgrades old data to the latest format so the rest of the codebase only ever sees `CURRENT_VERSION`.

To evolve the schema:

1. Bump `CURRENT_VERSION` in `src/persistence/migrate.ts`.
2. Write a migration function (e.g., `migrateV1toV2(data)`) that transforms the old shape to the new one. This is a pure function on a plain object — no imports from the type system needed.
3. Append it to the `migrations` array in the same file.
4. Add a test in `src/persistence/migrate.test.ts` that feeds old-format data through `migrate()` and checks the output.

The `validateScenario()` function in `SaveLoadButtons.tsx` calls `migrate()` before structural validation, so all load paths (JSON file, `?s=` URL) go through migration automatically.

Existing shared URLs and saved JSON files will keep working as long as the migration chain is maintained. Never remove a migration function — they're cumulative.

**Test requirements for migrations:**

- `migrate.test.ts` — unit tests: each migration must have a test that feeds old-format data through `migrate()` and asserts the output shape. Add one per version bump.
- `migrate.property.test.ts` — property tests that verify structural invariants across all migrations: output passes `validateScenario`, idempotency (`migrate(migrate(x)) ≡ migrate(x)`), no input mutation, and migration chain length matches `CURRENT_VERSION - 1`. These tests do not need updating when adding a new migration — they run automatically against whatever the current chain is.
- `url-codec.property.test.ts` — property tests for the URL codec round-trip (`decode(encode(x)) ≡ x`). Update if the codec format changes.

### When Adding a New Component Type

1. Define its config type in `src/types/configs.ts` and add it to the `ComponentConfig` union in `src/types/components.ts`.
2. Implement the `SimComponent` interface in a new file under `src/engine/components/`.
3. Write unit tests in a matching `.test.ts` file.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [marcbrooker/stability-sim](https://github.com/marcbrooker/stability-sim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
