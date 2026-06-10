---
trigger: always_on
description: > Mission, scope, and values: see [`MISSION.md`](./MISSION.md). The rules below are how we build it.
---

# Engineering Rules

> Mission, scope, and values: see [`MISSION.md`](./MISSION.md). The rules below are how we build it.

`CLAUDE.md` is a symlink to `AGENTS.md` at every level — same content, two filenames so any agent finds it. Per-package `AGENTS.md` files refine these rules for a specific package; they may add detail but **must not contradict this file**. When in doubt, the root wins.

Every PR is measured against the rules below. A change that violates an architectural principle doesn't land — we question the change before we question the principle.

> **Enforcement note.** Some rules below are enforced by tooling today (Biome formatter, fork harness in `@morpho-org/test`, Changesets generation). Most are **review-time conventions** that humans and reviewing agents apply: JSDoc on every export, layered-import bans, the §2 forbidden-patterns list (Biome's `noExplicitAny` is warn-level, `noParameterAssign` is disabled, and there's no rule banning `as unknown as` / `@ts-ignore` / async-in-actions / framework imports / mocked viem clients on RPC paths), changeset-gates-CI, full coverage thresholds. Where a rule isn't backed by an automated check, treat it as binding regardless — wiring CI gates is tracked separately.

> **Review personas.** The review-time conventions above are applied at PR review by specialized personas under [`.agents/personas/`](./.agents/personas/), invoked by the `/pr-review-{ci,gh,local}` slash commands. See [§10](#10-review-automation--cirelease-security) for the full inventory and the CI/release rules they anchor. When a rule below changes, the matching persona's bullet must change with it — the backlinks on each section name the personas to update.

---

## 1. Architecture

The three pillars: **layering, modularity, testability**. Everything else (types, API, tests, docs) flows from getting these right.

### Layering: one direction, no exceptions

`Client → Entity → Action`. Dependencies point one way. Lower layers never reach back into higher ones.

| Layer | Reads state? | Encodes calldata? | Async? | Returns |
| --- | --- | --- | --- | --- |
| **Client** | no | no | no | factory for entities |
| **Entity** | yes (RPC) | no | yes | lazy `{ buildTx, getRequirements }` |
| **Action** | no | yes | **no** | deep-frozen `Transaction` |
| **Helpers** | no | encode-only helpers (ABI-only); validators and constants are pure | no | new objects |

Cross-layer leaks (entities encoding calldata, actions reading state, helpers depending on entities) are an API design failure, not an implementation detail. Redesign the boundary; do not add a shortcut.

### Modularity: one reason to exist

- Every package has one clear job. If a package needs a paragraph to describe, split it.
- Every module has one responsibility. Files grow by *adding* exports of the same kind, never by stretching scope.
- **Single source of truth** per concept: one place per ABI, one place per address registry, one place per error class. Duplication is a refactor, not a feature.
- Framework adapters live in explicitly named packages (`*-wagmi`, `*-viem`). Core packages stay framework-free.
- Public API = barrel re-exports from `src/index.ts`. No deep imports across packages, ever.

### Testability: pure cores, I/O at the edge

- Network, clock, randomness, signing, env, and file I/O live **only** in boundary modules.
- Inner functions are deterministic and unit-testable without mocks. If a function is hard to test, its shape is wrong.
- Dependency injection through arguments, not globals or singletons. The viem client is always passed in.
- "Pure where possible" is a design rule, not a stylistic preference — every pure function we write is one fewer function we have to fork-test.

### Stateless, immutable, composable

- `morphoViemExtension()` rides on top of a viem client the integrator owns, exposing a stateless `morpho` namespace under `client.morpho` plus readonly options. No `init()`, no cache, no warm-up — those couple us to a host runtime and break statelessness.
- Every returned `Transaction` is `deepFreeze`d. Public fields are `readonly`. Helpers return new objects, never mutate inputs.
- Small primitives that combine. No kitchen-sink helpers; no boolean-prop explosions.
- Prefer early returns over deep nesting — guard clauses first, happy path last.

> Applied by personas: [`module-api-architecture`](./.agents/personas/module-api-architecture.md), [`morpho-protocol`](./.agents/personas/morpho-protocol.md) (protocol routing + ABI/address source of truth), [`web3-security`](./.agents/personas/web3-security.md) (Action-layer purity).

---

## 2. What does not land in a PR

A scannable list of patterns reviewers reject. Most are review-only today (per the Enforcement note); treat them as binding regardless. Reviewers cite this section by number.

1. `any`, `@ts-ignore`, `@ts-expect-error` (without an issue link and a deletion plan), and `as unknown as` outside test files. Test files may use `as unknown as` for narrow fixtures or test-only adapters when a real object would obscure the behavior under test.
2. `throw new Error(...)` from SDK source. Every failure mode is a named, exported class.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [morpho-org/sdks](https://github.com/morpho-org/sdks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-10 -->
