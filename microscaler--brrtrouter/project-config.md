---
trigger: always_on
description: > **Desktop dev environment** — before doing anything in this repo, read the
---

# BRRTRouter — agent rules

> **Desktop dev environment** — before doing anything in this repo, read the
> Microscaler-wide topology brief. It explains that you are on a Mac but the
> code lives on `ms02` (NFS), where commands execute for this environment, how
> the Kind cluster and vLLM fit in, and the network constraints behind the SSH
> tunneling. Do not duplicate its contents here — link to it. If reality drifts,
> fix the canonical doc, not this copy.
>
> - GitHub: [`cylon-local-infra/docs/desktop-dev-environment.md`](https://github.com/microscaler/cylon-local-infra/blob/main/docs/desktop-dev-environment.md)
> - On ms02 NFS: `~/Workspace/microscaler/cylon-local-infra/docs/desktop-dev-environment.md`

---

Strict operational rules for AI assistants working in this repository. **Knowledge about how BRRTRouter works is in [`llmwiki/`](./llmwiki/), not here.** This file only holds rules the agent must obey.

---

## Before you do anything

1. Read [`llmwiki/index.md`](./llmwiki/index.md) — the content catalog.
2. Read [`llmwiki/SCHEMA.md`](./llmwiki/SCHEMA.md) — wiki conventions and workflows.
3. Tail [`llmwiki/log.md`](./llmwiki/log.md) for recent context from other sessions.
4. Read the specific topic / entity / reference pages relevant to your work. Drill into prose `docs/` or source only when the wiki flags drift or a gap.
5. If the task is **scheduled / autonomous perf research**, read [`llmwiki/topics/auto-research-perf-loop.md`](./llmwiki/topics/auto-research-perf-loop.md) and the charter in [`auto-research/docs/PERF_CONTROL_SURFACE_AND_LOG.md`](./auto-research/docs/PERF_CONTROL_SURFACE_AND_LOG.md).

Sessions that skip this waste work. The wiki is the compounding artifact that makes knowledge persist across sessions — see [Karpathy's llm-wiki gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f).

---

## Repository shape

- Primary language: Rust (workspace with `brrtrouter`, `brrtrouter_macros`, and `examples/pet_store`).
- UI demo: SolidJS + Vite in `sample-ui/`.
- Generated code: `examples/pet_store/` is auto-generated; do not edit directly.
- Sibling repos (typical `microscaler/` checkout): **`../hauliage/`** (primary HTTP consumer + BFF) — [`docs/llmwiki/`](../hauliage/docs/llmwiki/). **`../lifeguard/`** (ORM + migrations) — [`docs/llmwiki/`](../lifeguard/docs/llmwiki/). Use [`llmwiki/topics/sibling-repos-and-wikis.md`](./llmwiki/topics/sibling-repos-and-wikis.md) for a responsibility split.

---

## Build, lint, test commands

### Recommended entry points (justfile)

- `just dev-up` / `just dev-down` / `just dev-status` — local dev environment (kind + Tilt).
- `just gen` — Regenerate `examples/pet_store/` from `examples/openapi.yaml`.
- `just build-ui` — Build SolidJS dashboard into `examples/pet_store/static_site`.

### Build

- `cargo build` — Build default workspace.
- `cargo build -p pet_store` — Build the generated pet store example.
- `cargo build --release` — Release build (used by Tilt sync).

### Format / lint

- `cargo fmt` — Format Rust (always run before committing).
- `cargo clippy --workspace --all-targets --all-features` — Lint. Configured deny-lints in `Cargo.toml`.

### Test

- `just test` — `cargo test -- --nocapture`.
- `just nt` — Fast parallel tests with nextest (recommended).
- `cargo test --test server_tests` — Single integration test module.
- `cargo test router::tests::test_route_matching` — Single test by name.
- `cargo test -- --ignored` — Ignored tests.

### Coverage / bench / profiling

- `just coverage` — Coverage report (must stay ≥80%).
- `just bench` — Criterion benchmarks.
- `just flamegraph` — Generate flamegraphs.

### UI (sample dashboard)

- `yarn install` (in `sample-ui/`).
- `yarn dev` — Dev server.
- `yarn build:petstore` — Build into `examples/pet_store/static_site`.

---

## Core rules the agent must obey

### 1. Do not edit generated code

`examples/pet_store/` is regenerated from `examples/openapi.yaml`. Any edit will be clobbered. Fix the spec or the template under `templates/` instead, then regenerate with `just gen`.

### 2. Follow Rust conventions

- `snake_case` for fns / modules, `CamelCase` for types, `SCREAMING_SNAKE_CASE` for constants.
- Group imports: std, external crates, local modules. Prefer explicit imports.
- `Result<T, E>` + `?` over `panic!` in library paths. No `unwrap()` / `expect()` in production (`clippy::unwrap_used` is deny).

### 3. Hot-path JSF-AV safety

- Avoid allocations on routing / dispatch hot paths (use `SmallVec`).
- Preallocate (`with_capacity`) when collections are needed.
- Keep dispatch paths deterministic; no panics.
- Unsafe is allowed but must be isolated and well-justified (document safety invariants).

### 4. Documentation expectations

- Public modules require `//!` module-level docs (overview, architecture, examples).
- Public fns / structs / enums / traits require `///` docs (purpose, args, returns, examples, panics / safety).
- Test modules have `//!` module docs explaining coverage and strategy.

### 5. Testing discipline

- Run `just nt` before submitting.
- Keep tests deterministic; avoid global state.
- Maintain ≥80% coverage; add tests for new behaviour.

---

## Commit discipline


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microscaler/BRRTRouter](https://github.com/microscaler/BRRTRouter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
