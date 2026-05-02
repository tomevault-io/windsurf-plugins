---
trigger: always_on
description: All agents and contributors working in this repository must treat `docs/engineering/design-language.md` as a global visual and frontend constraint.
---

# AGENTS.md

## Global Instruction

All agents and contributors working in this repository must treat `docs/engineering/design-language.md` as a global visual and frontend constraint.

That file defines the authoritative design language for this project.

## Required Rule

For any UI, frontend, preview, or visual implementation work:

- Read and follow `docs/engineering/design-language.md`
- Do not introduce conflicting visual language unless the user explicitly requests it
- Do not hardcode visual primitives that should come from shared design tokens
- Preserve the project's Modern Minimalist Glassmorphism + Clean UI direction

## Priority

If a task touches styling, layout, panels, controls, previews, or renderer-facing components, the design-language document is a hard constraint, not a suggestion.

Only direct user instruction can override it.

不允许写任何兼容性代码, 做任何兼容性迁移行为. 我们处于原型开发阶段.所有改进做breaking change.

## Quality Gate — Test Pipeline Must Pass

No implementation is considered complete until the repository test pipeline passes on the current branch.

### Mandatory Test Commands

```bash
# Regenerate deterministic generated tests
npm run test:generate

# Run the repository quality gate
npx vitest run

# Run real Electron Playwright journeys
npm run test:e2e

# Verify declared behavior assets are covered
npm run test:behavior-coverage
```

### Quality Compliance Rules

1. **All tests must pass.** If a test fails, the implementation is not done. Fix the code, not the test.
2. **Generated tests are part of the source of truth.** Run `npm run test:generate` before verification and do not hand-edit files under `tests/generated/`.
3. **Do not delete or skip failing tests** to make the suite green. Fix the underlying code, contract, topology, or behavior asset.
4. **Do not use `as any`, `@ts-ignore`, or `@ts-expect-error`** in any test file.
5. **Treat behavior coverage as a gate, not a report.** If critical behavior assets are declared but not verified/hardened as required, the work is incomplete.

### Test Architecture

The test suite is organized in four layers. Every layer must pass independently:

#### Tier 1: Unit Tests (`src/**/*.test.ts`)

Direct tests for individual modules with isolated mocks. Fast, no file system side effects.

- `src/core/project-session-manager.test.ts` — Project/session CRUD, recovery plans
- `src/core/state-store.test.ts` — JSON persistence read/write
- `src/core/webhook-server.test.ts` — HTTP endpoint acceptance
- `src/core/webhook-server-validation.test.ts` — All event validation rejection branches
- `src/core/session-runtime.test.ts` — Resume vs fresh-start command selection
- `src/core/session-runtime-callbacks.test.ts` — onData/onExit callbacks, default values, canResume branches
- `src/core/pty-host.test.ts` — PTY spawn, write, resize boundaries, dispose, exit cleanup
- `src/core/app-logger.test.ts` — Log file writing
- `src/main/preload-path.test.ts` — Preload path resolution, webPreferences config
- `src/extensions/providers/opencode-provider.test.ts` — OpenCode command building
- `src/extensions/panels/index.test.ts` — Panel registry
- `src/renderer/stores/workspaces.test.ts` — Pinia store hydrate/hierarchy/active cascading
- `src/renderer/app/App.test.ts` — Root component bootstrap/IPC mock/error handling
- `src/renderer/components/**/*.test.ts` — All Vue component tests

#### Tier 2: E2E Integration Tests (`tests/e2e/*.test.ts`)

Full pipeline tests using real file system, real HTTP requests, real Pinia stores. No module-level mocks except for Electron IPC.

- `tests/e2e/backend-lifecycle.test.ts` — Fresh start → multi-project → session CRUD → state persistence → restart recovery → webhook server → session runtime → provider commands
- `tests/e2e/frontend-store-projection.test.ts` — Real backend → Pinia hydrate → computed properties → active cascading → store-backend consistency
- `tests/e2e/error-edge-cases.test.ts` — Duplicate paths, orphan sessions, state corruption recovery, concurrent managers, rapid operations, path normalization
- `tests/e2e/provider-integration.test.ts` — Provider registry, command building, environment variables, sidecar file writing with real disk verification
- `tests/e2e/ipc-bridge.test.ts` — Simulated FakeIpcBus round-trip: renderer → preload → ipcMain → manager → response
- `tests/e2e/app-bridge-guard.test.ts` — App.vue behavior when window.vibecoding is undefined/partially defined/null responses
- `tests/e2e/main-config-guard.test.ts` — Static analysis: sandbox:false presence, IPC channel registration completeness, preload type contract

#### Tier 3: Generated Contract and Journey Assets (`testing/**/*.test.ts`, `tests/generated/**/*.spec.ts`)

These files define and validate the AI-first testing layer:

- `testing/contracts/*.test.ts` — Contract DSL invariants and generated metadata checks
- `testing/behavior/*.test.ts` — Behavior graph declarations and risk/coverage budget validation
- `testing/topology/*.test.ts` — Stable `data-testid` topology contracts
- `testing/journeys/*.test.ts` — Journey declarations that map behaviors to executable paths
- `testing/generators/*.test.ts` — Deterministic generator and behavior coverage logic

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bainianlaoyao/Stoa](https://github.com/bainianlaoyao/Stoa) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
