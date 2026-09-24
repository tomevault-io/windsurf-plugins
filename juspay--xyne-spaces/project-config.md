---
trigger: always_on
description: - Gauge + Playwright TypeScript automation suite.
---

# Xyne Automation - AGENTS.md

## What this package is

- Gauge + Playwright TypeScript automation suite.
- BDD style, not story-heavy. Goal: reliable executable behavior checks.
- Tests under `tests/`, run through Gauge, not Playwright `test()` files.

## Core principles

- **Self-contained** - no scenario-order or leftover-state dependence.
- **Deterministic** - same setup, same result.
- **Parallel-safe** - safe beside other scenarios.
- **Simple, small, clean** - readable specs, small steps, reusable concepts.
- **Validation-friendly** - reuse validated literals, names, patterns.

## LLM agent guidelines

When writing or suggesting tests, follow these rules:

### Think parallel-first

Every scenario may run simultaneously with every other. Design accordingly:

- Never assume execution order between scenarios.
- Never read state written by another scenario.
- Never mutate shared resources (baseline fixture, shared users) expecting other scenarios see change.
- If two scenarios touch same entity, use separate instances or prove safe under concurrent access.

Before writing any test, ask: "Would this break if another scenario runs same time?" If yes, redesign.

### Use fixture data, not test-created data, for expected values

Tests derive expected values from fixtures and user catalog — not from data created during execution.

- **Right**: Store channel name from baseline fixture via `ensuring channel <channelAlias> exists in fixture for user <userAlias>`, verify against stored value.
- **Wrong**: Create channel with hardcoded name, assert that exact name appears. Couples assertion to creation step, breaks on retry (unique suffixes change).
- **Wrong**: Generate value in one step, pass through global state, assert in another — hides expected value, makes failures harder to diagnose.

Baseline fixture and user catalog = source of truth. Use `user:<alias>.<field.path>` interpolation for stored data. If test needs data not in fixtures, create through proper fixture/ensure flow, store it, reference stored version — never inline literals as both creation input and assertion target.

### Suggest fixture-based approaches to users

When user asks to write test needing pre-existing data (project, channel, membership, etc.):

1. Check if baseline fixture already provides what's needed.
2. If yes, suggest using baseline data instead of creating new entities.
3. If no, suggest extending fixture system or using ensure/store pattern — not ad-hoc creation with hardcoded values.
4. Explain why: parallel safety, retry resilience, single source of truth.

### Expected vs actual in assertions

- Expected value = what system should produce, derived from fixture/stored data.
- Actual value = what UI or API returned.
- Always make distinction clear in assertion steps. Use stored interpolation (`user:admin-1.projects.project-1.name`) for expected values, page/API queries for actual values.

## Package layout

- `config/` - runtime config from env.
- `lib/` - shared utilities (logger with `gaugeLogger` and `baselineLogger` exports).
- `scripts/run-gauge.ts` - Gauge runner wrapper.
- `scripts/validation/` - filename and literal validation.
- `tests/01_api/` - API scenarios.
- `tests/02_ui/` - UI scenarios.
- `tests/03_e2e/` - E2E scenarios.
- `tests/shared/` - shared steps, runtime store, browser manager, literal validators, report artifact snapshots, suite hooks.
- `fixtures/` - static automation fixtures: user catalog, fixture helpers, baseline fixture.
- `env/default/` - Gauge runtime properties.
- `reports/`, `.gauge/` - runtime artifacts, gitignored. Gauge runs create `reports/<9-char-commit-hash>-<sequence>/` folders with `html-report/`, `gauge.log`, `run-metadata.json`, and runner folders (`runner/<n>/` when Gauge exposes worker ids, otherwise `runner/pid-<pid>/`) containing `runner.log` plus `context.json` snapshots.

## Developer commands

```bash
# Run all tests
npm test

# Run by test area
npm run test:api      # tests/01_api only
npm run test:ui       # tests/02_ui only
npm run test:e2e      # tests/03_e2e only

# Run a single spec file directly via Gauge
npx gauge run tests/01_api/01_health/01_health.spec

# Run a single scenario by name
npx gauge run --tags "scenario-name" tests/path/to/file.spec

# Full validation (required for all non-doc code changes)
npm run validate      # biome ci + biome format check + filename validation + literal validation

# Individual validation steps
npm run check         # biome lint + assist
npm run check:fix     # biome lint + assist with auto-fix
npm run format        # biome format with auto-write
npm run format:check  # biome format check only
npm run validate:filename   # numbered folders/specs, no .js files
npm run validate:literals   # browser names, assertion hooks in step defs
```

`validate` chain: `biome ci` -> `biome format` -> `validate:filename` -> `validate:literals`. All four must pass.

## Execution rules

- Run smallest relevant test scope for changed area.
- Run full `npm run validate` for any non-doc code change.
- If change affects shared steps, validators, config, or runtime behavior, validation mandatory.

## BDD structure rules


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [juspay/xyne-spaces](https://github.com/juspay/xyne-spaces) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
