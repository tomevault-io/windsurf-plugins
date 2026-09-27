---
trigger: always_on
description: - These instructions apply to the entire repository unless a closer `AGENTS.md` or
---

# Repository Working Agreements

## Scope and precedence

- These instructions apply to the entire repository unless a closer `AGENTS.md` or
  `AGENTS.override.md` supplies narrower instructions.
- Preserve existing behavior unless the task explicitly authorizes a behavior change.
- If requested work conflicts with these instructions, stop and report the conflict.

## Canonical commands

- Install dependencies: `npm ci`
- Fast validation: `npm run check:fast`
- Full required validation: `npm run check`
- Disposable-database integration validation: `npm run test:integration`
- Guardrail verification: `npm run test:guardrails`
- Submitted-diff grading: `git diff --no-renames --unified=0 <base> <head> | node guardrail-evals/run.mjs --stdin --base-ref <base> --head-ref <head>`

Use the Node version supported by `package.json` and npm with the committed lockfile.
Do not substitute a different package manager. Completion requires observed command
results; configuration or an existing test file is not evidence that a check passed.

Write-capable integration commands, including `npm run test:integration` and
`npm run test:release:database`, require guarded disposable test credentials and a
verified test-only database target. [DB-WRITE] Never run them using a general
development, staging, or production `DATABASE_URL`, and never infer authorization to
run them from a request to run the ordinary test suite.

## Change discipline

- Make the smallest coherent change that satisfies the requested behavior.
- Keep unrelated cleanup out of the change.
- Follow existing boundaries among domain logic, adapters, MCP tools, HTTP transport,
  commands, and browser UI. Do not introduce an abstraction until it has a concrete
  current use.
- Explain any new production dependency and why existing dependencies or Node
  capabilities are insufficient.
- Do not edit generated `dist/` output or generated browser bundles directly; change
  their source and rebuild.

## Type and boundary safety

- [TG-1] Keep TypeScript strict. Do not weaken compiler options or exclude failing
  production, UI, test, or script paths to obtain a passing result.
- [TG-2] Do not introduce `@ts-ignore`, broad `eslint-disable`, unsafe `any`, or
  unexplained double casts. Prefer a real type or boundary fix. A narrow
  `@ts-expect-error` or rule-specific suppression requires an adjacent reason and
  evidence that the exception is intentional.
- Treat MCP, HTTP, environment, file, CLI, database, and embedding-provider values as
  untrusted. Parse and bound them at entry before constructing trusted domain values.
- Make invalid lifecycle states difficult to represent. Handle closed states
  exhaustively and preserve revision and approval invariants.

## Control flow, resources, and failures

- Keep control flow locally understandable. Split code when a function mixes
  orchestration, policy, I/O, and transformation.
- Bound retries, polling, pagination, queues, concurrency, recursion, request bodies,
  import files, result sets, and retained histories.
- Define timeout, cancellation, backoff, idempotency, and terminal failure behavior
  where relevant.
- Await or intentionally supervise every promise. Preserve useful failure context and
  never silently swallow errors.
- Release timers, listeners, streams, handles, transactions, and temporary resources
  on success, failure, and cancellation.
- Keep database privilege separation intact. Changes to approval functions,
  `SECURITY DEFINER` SQL, migration checksums, remote HTTP binding, or gateway trust
  assumptions are critical changes.

## Behavioral evidence

- [TG-7] Add or update deterministic tests for changed behavior. Do not delete, skip,
  focus, loosen, or inflate timeouts in tests merely to obtain a passing result.
- For bug fixes, add a regression case that fails without the fix.
- Cover the intended path and relevant invalid, boundary, timeout, cancellation,
  duplicate/replay, and dependency-failure paths.
- Control time, randomness, network, and external state. Tests in the fast and full
  canonical paths must not require production credentials.
- Run the narrowest relevant checks during implementation and `npm run check` before
  completion. Report any checks not run and why.

## Protected control plane

Treat these as protected control-plane surfaces:

- [TG-10] `AGENTS.md`, nested agent instructions, `guardrail-evals/`, and waivers;
- TypeScript, lint, formatting, test, coverage, package, and build configuration;
- `.github/workflows/`, required-check definitions, ownership, and release files;
- dependency manifests and lockfiles;
- authorization, database roles, migrations, destructive tooling, public contracts,
  runtime schemas, and generated-code sources.

Do not silently weaken a protected surface. Report material changes separately,
including their reason and the evidence protecting them. Do not:

- change a required check to advisory or omit `npm run check` from required CI;
- reduce checked paths, test scope, or assertion strength;
- add unexplained suppressions, skips, focus markers, timeout increases, snapshot
  churn, or lockfile churn;
- change authorization, destructive behavior, public contracts, migration semantics,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [knoxgraeme/tieline](https://github.com/knoxgraeme/tieline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
