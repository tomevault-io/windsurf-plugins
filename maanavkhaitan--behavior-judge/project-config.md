---
trigger: always_on
description: Orientation for coding agents (and humans) working on `behavior-judge`. Verify
---

# Agent context for this repo

Orientation for coding agents (and humans) working on `behavior-judge`. Verify
details against the code if much has changed since 2026-08.

## 1. The repo in one paragraph

`behavior-judge` is a companion tool to the
[Agent Behavior standard](https://github.com/braintrustdata/agentbehavior): specs
(`BEHAVIOR.md`) describe expected agent conduct in prose and deliberately say nothing
about how to judge a trajectory against them. This package closes that gap — it compiles
a spec into a **checked-in YAML judge IR** where deterministic event-pattern predicates
do most of the judging for free, and the LLM is confined to three narrow,
individually-validated jobs: semantic triggers, semantic checks, and one confirmation
call per predicate `false`. Every verdict traces to a verbatim spec `quote` plus
event-ID citations. The package was extracted (history-preserving) from a fork of the
standard's monorepo; it is Apache-2.0, and the example spec, tax fixture data, and
gateway client derive from that repo's examples.

## 2. Tooling and conventions

- **pnpm** (`pnpm@10.33.0` via `packageManager`) + **vite-plus (`vp`)**: `pnpm build`
  (`vp pack`: build, dts, esm), `pnpm test` (`vp test --run`, vitest-compatible; import
  from `"vite-plus/test"`), `pnpm check` (`vp check [--fix]`, fmt + lint). Run
  `pnpm exec vp check --fix` before committing.
- TypeScript: `NodeNext` + `.js` import extensions, `strict`,
  `exactOptionalPropertyTypes`, `noUncheckedIndexedAccess`. Type `module`.
- CLI pattern (`src/cli.ts`): `main(argv, deps?): Promise<number>`,
  `process.stdout.write`, `pathToFileURL` entry guard, `node:util parseArgs`, injectable
  deps for tests, `captureMain` stdout/stderr spy pattern in tests.
- **Gotchas:**
  - `pnpm-workspace.yaml` exists only to mark this directory as its own pnpm root —
    without it, a checkout nested under another pnpm workspace installs to the wrong
    root. Don't delete it.
  - `pnpm exec tsc --noEmit` emits a TS6059 rootDir complaint about `vite.config.ts` —
    long-standing quirk inherited from the monorepo; harmless (`vp` builds fine).
  - The repo lives under the `MaanavKhaitan` GitHub account; if `gh` has multiple
    accounts logged in, `gh auth switch -u MaanavKhaitan` before pushing (and switch
    back after).

## 3. CLI surface

```
behavior-judge generate  <behavior-path> <trajectory.json ...> [--update <ir.yaml>] [--out <file>] [--model <m>] [--no-web]
behavior-judge judge     <ir.yaml> <trajectory.json ...> [--json] [--model <m>] [--no-verify] [--no-web]
behavior-judge calibrate <ir.yaml> <trajectory.json ...> [--json] [--model <m>] [--no-verify]
```

Exit codes: `judge` 0 on successful run; `calibrate` 1 on any expected/actual
disagreement (CI gate); `generate` 1 if the user declines the final confirm. Errors → 1.
`generate --update <ir.yaml>` is the diff-scoped re-interview after a spec edit (§10);
`--out` then defaults to the `--update` path. The browser is the default frontend:
`generate` serves the interview to a browser (§10a) unless `--no-web` picks readline,
and combines with `--update` (the update interview adds two step kinds the page
renders, §10a); `judge` serves the report to a browser (§10b) unless `--no-web` or
`--json` picks the terminal (json implies the terminal report). Explicit `--web` is
still accepted as an opt-in; it rejects `--json` (one format at a time) and `--no-web`,
and `calibrate` does not support the browser yet (it always reports in the terminal;
explicit `--web` errors).

## 4. Source map (`src/`, dependency order)

Layout: the judging engine lives in `src/core/`, the spec→IR authoring flows in
`src/interview/`, and the two web frontends in `src/web/`; the entry points
(`index.ts`, `cli.ts`) and the CLI-only `env.ts` stay at the `src/` root. Tests are
colocated with their sources. Dependencies flow one way — web → interview → core —
and the root entry points import all three.

**`src/core/` — the judging engine:**

- `trajectory.ts` — `TrajectoryEvent`/`AgentTrajectory`/`ExpectedBehaviorJudgment`/
  `TrajectoryCase` + `loadTrajectoryFile` (accepts bare trajectory, `{trajectory,
expected}` wrapper, or array of either; rejects duplicate event IDs).
- `spec.ts` — `loadBehaviorSpec`: minimal BEHAVIOR.md loader (file or directory path,
  frontmatter `name`/`description`, markdown body). Deliberately NOT a full validator —
  lint specs against the standard with the upstream `agentbehavior` CLI.
- `ir.ts` — IR types + strict `parseIr` (fail-fast, path-labeled errors like
  `metaBehaviors[0].checks[1].quote`), `serializeIr`, `foldBehaviorVerdicts`,
  `behaviorVerdictToScore`.
- `predicates.ts` — pure deterministic core: `matchesEvent`/`matchesAny`/`findMatches` +
  `evaluatePredicate`. No LLM, no IO.
- `text.ts` — `flattenWhitespace`/`clip`, the whitespace normalization shared by quote
  matching (update.ts), evidence rendering (generate.ts), and web payloads.
- `gateway.ts` — Braintrust Gateway client + JSON helpers + `completeJsonWithRetry`
  (retry-once-with-error-appended) + `JudgeCompletion` type.
- `semantic.ts` — the scoped LLM check: one system prompt, `parseSemanticResult`
  (verdict enum; `na_reason` iff `na`; ≥1 citation whose event IDs must exist), and two

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MaanavKhaitan/behavior-judge](https://github.com/MaanavKhaitan/behavior-judge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
