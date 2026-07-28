---
trigger: always_on
description: A CLI and library with two surfaces. The **audit surface** (v10, the
---

# Swarm Orchestrator

A CLI and library with two surfaces. The **audit surface** (v10, the
headline) is `swarm audit <pr | --diff-file | --diff-stdin>`: it walks a
PR diff through a pluggable cheat-detector registry (test relaxation,
mock-of-hallucination, assertion strip, no-op fix, and seven more),
fingerprints the AI agent that wrote the PR, optionally emits a
CycloneDX-ML or SPDX 3.0 AI-Profile AIBOM, and posts a deterministic
finding back to the PR. The **orchestrator surface** (v8) compiles a
natural-language goal into a typed contract and runs it through a
falsification-gated pipeline (extractor → session → predicate-runner →
falsifier → verifier), admitting only patches that satisfy every
obligation. v6 (the verified-branch pipeline that wrapped third-party
coding-agent CLIs) was removed in v9.0.0; pin to `8.0.x` if you still
depend on it.

**The architectural rule:** nothing reaches `main` without passing
verification end-to-end. Don't introduce a merge path that bypasses
`verifyObligation` / `postMergeVerify`.

## Stack

- Node.js ≥ 20 (engines-enforced). CI runs 20 and 22.
- TypeScript strict mode, `target: ES2022`, `module: commonjs`, `exactOptionalPropertyTypes: true`.
- CommonJS. `require()` is the native import form. `@typescript-eslint/no-require-imports` is intentionally disabled.

## Commands

| Command | What it does |
|---|---|
| `npm run build` | `tsc -p tsconfig.build.json`, then `chmod 0755 dist/src/cli.js`. Runs `clean` first via `prebuild`. |
| `npm test` | Builds, then `mocha --recursive 'dist/test/**/*.test.js'`. |
| `npm run test:ci` | Mocha without a build (CI pre-builds). |
| `npm run typecheck` | `tsc --noEmit -p tsconfig.build.json`. |
| `npm run lint`, `npm run lint:fix` | ESLint on `src/**/*.{ts,tsx}` and `test/**/*.ts`. |
| `npm run format` | Prettier write. `format:check` exists but the codebase has historical drift; do not rely on it as a gate. |
| `npm start` | `node dist/src/cli.js`. |
| `npm run oracle:build` | Build the defect-injection oracle corpus under `benchmarks/oracle-corpus/` (deterministic). |
| `npm run benchmarks:baseline` | Freeze the pre-upgrade detector + judge metrics under `benchmarks/baselines/pre-upgrade/`. |
| `npm run benchmarks:oracle` | Score detector recall and judge-primary recall on the oracle corpus. |
| `npm run calibrate:judge` | Score every judge prompt version and pick the default. |
| `npm run oracle:evasion` | Run the evader stack and emit survival curves. |
| `npm run benchmarks:full` | Regenerate the whole oracle pipeline and `COVERAGE.md`. |
| `npm run agent-incidence:full` | Fetch agent-attributed PRs, audit, dual-arbiter classify, render `benchmarks/real-prs/agent-corpus/INCIDENCE-REPORT.md`. |
| `npm run block-eligibility:full` | Recalibrate block triggers against revert history; recompute `benchmarks/real-corpus/block-eligibility.json`. |

Before any PR: `npm test`, `npm run typecheck`, then a descriptive commit. The LOC budget gate (`scripts/loc-budget-gate.sh`) runs in CI against `evidence/loc-budget.txt`.

## Where things live

- `src/cli.ts`: top-level CLI dispatcher (`audit`, `run`, `compile`, `resume`, `stats`, `doctor`, `init`, `v8`). `audit` is the v10 user-facing verb; `run` and friends are the v8 orchestrator surface.
- `src/cli/v8/`: per-subcommand handlers for both surfaces.
  - `src/cli/v8/audit-handler.ts`: `swarm audit` dispatcher; resolves `--pr` / `--diff-file` / `--diff-stdin`, walks the cheat-detector registry, renders the comment, optionally emits the AIBOM.
  - `src/cli/v8/pr-fetch.ts`: PR-ref → unified-diff resolver used by `swarm audit --pr`.
  - `src/cli/v8/init-handler.ts`: `swarm init` scaffolding.
  - `src/cli/v8/doctor-handler.ts`: `swarm doctor` diagnostics (with `--fix`).
  - `src/cli/v8/run-handler.ts`: `swarm run` with auto-discover and preset integration.
- `src/audit/`: the v10 audit surface.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [moonrunnerkc/swarm-orchestrator](https://github.com/moonrunnerkc/swarm-orchestrator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
