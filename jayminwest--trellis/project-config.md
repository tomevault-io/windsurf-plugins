---
trigger: always_on
description: This file is the canonical entry point for AI coding agents working in
---

# AGENTS.md

This file is the canonical entry point for AI coding agents working in
`trellis`, following the [agents.md](https://agents.md) convention. For the
full design record see [`SPEC.md`](SPEC.md); for ecosystem context see
[`CLAUDE.md`](CLAUDE.md).

## Mission

`trellis` — a **deterministic, offline-by-default sloppiness audit** for
TypeScript/TSX workspaces. It parses source with the TypeScript compiler API
and measures structural debt — **complexity, structural erosion, duplication,
and import cycles** — plus a separate, non-scoring inspection of safeguard
configuration (hooks and check wiring). Each run emits a versioned report
with a **0–100 sloppiness index where lower is better** (not a percentage of
bad code; infrastructure cannot offset it), raw metrics, traceable score
contributions, ranked hotspots, and safeguard evidence.

Three invariants define the product (SPEC §1):

1. **No-model execution.** No audit path — CLI, SDK, fleet, or CI — spawns an
   agent, calls a model, or consumes model-derived grading.
2. **Offline and zero-footprint by default.** The first audit needs neither
   Git nor credentials, a database, network, or installed project
   dependencies; it writes nothing unless the operator asks.
3. **One core, every surface.** Local, fleet, and CI runs exercise the same
   deterministic core; CLI and SDK are thin pass-throughs.

> **Optional quality-evidence providers (plan `pl-43c5`, SPEC §16).**
> jscpd and dependency-cruiser run as explicitly opt-in, unscored evidence.
> Knip supplies contextual reachability candidates; SonarJS remains deferred.
> Native analysis remains the authoritative scoring basis. Current setup,
> trust boundaries and migration examples: [`docs/quality-evidence.md`](docs/quality-evidence.md).
> Executed platforms and remaining acceptance: [`docs/provider-acceptance.md`](docs/provider-acceptance.md).

trellis is part of [os-eco](https://github.com/jayminwest/os-eco), the AI agent
tooling ecosystem. It is the **measurement surface**: it gives the fleet an
objective, reproducible read on structural code health. trellis mirrors the
warren/burrow Bun + TypeScript-strict + Biome + SQLite stack and dogfoods its
own audit (SPEC §14).

## Commands

All commands run from the repo root unless noted. `Bun` must be on PATH.

```bash
bun install                   # install dependencies
bun test                      # run all tests
bun test <path/to/file>       # run a single test file
bun run lint                  # biome check --error-on-warnings .
bun run lint:fix              # biome check --write --error-on-warnings .
bun run typecheck             # tsc --noEmit
bun run check:all             # full quality-gate suite (see below)
bun run verify                # alias for check:all (agent-facing entry point)
bun run check:coverage        # tests + coverage ratchet
bun run test:ci               # bun test with junit + coverage reporters
```

trellis ships a CLI (`trellis`, bin `./src/cli/main.ts`). The deterministic
surface (SPEC §12) includes optional fleet/history and separate canonical
standards/drift inspection:

```bash
trellis audit <path>          # measure + score one workspace; print the sloppiness report
                              #   [--json|--md] [--out <file>] [--baseline <report.json>]
                              #   [--config <file>] [--history] [--db <path>]
trellis compare <a> <b>       # compare two saved report artifacts (no audit)
trellis fleet                 # audit every target in targets.yaml through the same core
                              #   [--history] [--db <path>] (drift rides along, never scored)
trellis report                # sloppiness history from SQLite
trellis drift <repo-path>     # canonical-config drift only (separate, unscored)
trellis standards             # show canonical manifest + versions
trellis brand <repo-path>     # static os-eco CLI brand check (docs/brand-standard.md)
trellis guide cleanup         # print bundled, read-only cleanup guidance
```

`--json` / `--md` switch terminal output to machine/report shapes. The
default `audit` run is **stateless** — no database, no report files — unless
`--history` / `--out` ask.

### Exit codes (SPEC §9)

Every command exits `0` clean, `2` when a policy trips (the report is still
emitted to stdout; the reason goes to stderr), or `1` on an operational
error (the command could not run). On the deterministic surface the policy
is **declarative**: the policy block of the workspace's trellis.yaml (SPEC §6.5 — max
index, metric budgets, score regression, new-finding kinds) gates `audit`
and `compare`, and an incompatible `compare` pair fails closed. `EXIT`
lives in `src/cli/output.ts`; the assessment is core (`assessPolicy` in
`src/compare/policy.ts`), so the CLI and SDK gate identically. `fleet`
gates on each target's own declarative policy result plus per-target
operational failures (`assessFleet` in `src/fleet/assess.ts`); canonical
drift rides along as a separate, non-scoring capability and never gates.
The separate `drift` command retains `--fail-on drift|none` for canonical
configuration policy.

### Programmatic SDK (`src/client/`)

`src/client/index.ts` exposes `audit` / `compare` / `fleet` / `report` over

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jayminwest/trellis](https://github.com/jayminwest/trellis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
