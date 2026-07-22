---
trigger: always_on
description: This file is the canonical entry point for AI coding agents working in
---

# AGENTS.md

This file is the canonical entry point for AI coding agents working in
this repo, following the [agents.md](https://agents.md) convention. It
mirrors the essentials from [`CLAUDE.md`](CLAUDE.md); when the two
disagree, `CLAUDE.md` is authoritative and this file should be updated
to match.

## What this project is

Warren is a self-hostable control plane for ephemeral cloud agents.
Point it at a GitHub repo, pick an agent, write a prompt; warren spawns
the agent inside a sandbox (burrow), streams events to the UI, lets the
user steer mid-run, then pushes the workspace branch. One container,
one volume, one HTTP API, one UI.

The fresh-install path is standalone: the built-in `claude-code` agent
ships inline (`src/registry/builtins/`). The bundled os-eco data-plane
features (`canopy`, `mulch`, `seeds`, `sapling`, `plot`, `plan-run`) are
**opt-in** and light up when their config / directories are present —
see `CLAUDE.md` and [`SPEC.md`](SPEC.md) §1 / §11 for the full framing.

The runtime substrate is [burrow](https://github.com/jayminwest/burrow);
warren and burrow are co-tenanted inside the container and share a unix
socket. **Before touching anything that crosses the warren↔burrow
boundary** (`src/supervisor/main.ts`, `src/burrow-client/`,
`docker-compose.yml` security flags), read `../burrow/SPEC.md` and the
"Relationship to burrow" section of `CLAUDE.md`.

## Tech stack at a glance

- **Runtime:** Bun (runs TypeScript directly, no server build step)
- **Language:** TypeScript, strict mode (`noUncheckedIndexedAccess`, no `any`)
- **Lint/format:** Biome (`--error-on-warnings` — warnings fail CI)
- **Storage:** SQLite via `bun:sqlite` (Postgres optional, see SPEC §11.J)
- **HTTP:** `Bun.serve` serves both the JSON API and the SPA
- **UI:** React + Vite + Tailwind + shadcn-style components, in
  `src/ui/` as the `@os-eco/warren-ui` package, built into `src/ui/dist/`
- **Sandbox primitive:** burrow (HTTP over a unix socket)

## Build & test commands

From the repo root:

```bash
bun test                      # Run all tests
bun test src/foo.test.ts      # Run a single test file
bun run test:ci               # bun test --reporter=junit -> test-results/junit.xml
bun run test:coverage         # bun test --coverage (text + lcov -> coverage/)
bun run check:coverage        # tests + coverage + ratchet enforcement
bun run report:test-timing    # print slowest suites/tests from junit.xml
bun run report:quality-metrics # print code-quality metrics summary (coverage + complexity + ratchets)
bun run lint                  # biome check --error-on-warnings .
bun run typecheck             # tsc --noEmit
bun run build:ui              # cd src/ui && bun install && bun run build
```

CI (`.github/workflows/ci.yml`, warren-cec7) runs `bun run test:ci` instead
of bare `bun test` so every PR emits `test-results/junit.xml`, then runs
`bun run report:test-timing` to dump a slowest-suite/slowest-test summary
into the GitHub Actions step summary, then `bun run report:quality-metrics`
(warren-5b95) appends a consolidated code-quality panel — coverage % vs
floors, complexity grandfather counts, file-size + debt-marker ratchet
status, and bundle-size headroom — to the same summary, and uploads the JUnit XML as the
`bun-test-junit` artifact for offline analysis (regression triage, perf
ratchets, etc.). `test-results/` is gitignored — it's a build artifact.

UI-only (its own package):

```bash
bun run ui:dev                # vite dev server
bun run ui:install            # cd src/ui && bun install
```

## Quality gates

Run all checks before committing — warnings count as failures:

```bash
bun run check:all     # or its agent-facing alias: bun run verify
```

`check:all` is the os-eco canonical quiet runner (`scripts/check-all.ts`,
byte-identical to `../templates/l5-toolkit/scripts/check-all.ts` at the
os-eco root — never edit it in place). It prints one aligned status line
per gate and a `12/12 gates passed` tally; on failure it shows parsed
failure signatures plus a `re-run: bun run <gate>` hint
(`CHECK_ALL_VERBOSE=1` streams full output, `--bail` stops early).
Warren's resolved manifest, in order: `lint`, `typecheck`,
`check:agents`, `check:dups` (jscpd), `check:deps`, `check:size`,
`check:debt`, `check:bundle-size`, `gen:docs:check`, `gen:openapi:check`,
`check:coverage` (tests + coverage ratchet), and `check:ci-parity` —
the same set CI enforces (see `.github/workflows/ci.yml`; escape
hatches live in `scripts/ci-parity-config.json`). Do not merge with
lint warnings; fix at write time or promote to error in `biome.json`.

Details on the additional checks:

- **`check:size`** (warren-4553) — enforces a per-file line-count
  budget. New `.ts`/`.tsx` files under `src/` and `scripts/` must stay
  ≤ 500 lines; existing oversized files are grandfathered in
  `scripts/file-size-budgets.json` and may not grow past their frozen
  ceiling — the ratchet only goes down. Biome's
  `noExcessiveLinesPerFunction` rule (also 500-line cap) enforces the
  same budget at the function level, with the same baseline exceptions
  called out in `biome.json`'s `overrides`.
- **`check:debt`** (warren-7f2b) — scans `src/` and `scripts/`
  `.ts`/`.tsx` for `TODO` / `FIXME` / `HACK` / `XXX` and fails if any

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jayminwest/warren](https://github.com/jayminwest/warren) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
