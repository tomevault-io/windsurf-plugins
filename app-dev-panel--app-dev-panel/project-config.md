---
trigger: always_on
description: A passing `.` is the only acceptable result. **Every** other PHPUnit marker — `S` (skipped),
---

# ADP — Application Development Panel

## Zero Tolerance — Skipped, Risky, Deprecated, Warning Tests Are FAILURES

A passing `.` is the only acceptable result. **Every** other PHPUnit marker — `S` (skipped),
`I` (incomplete), `R` (risky), `D` (deprecation), `N` (notice), `W` (warning) — is a FAIL.
All `phpunit.xml.dist` files set these attributes to `true` and they **must never be flipped off**:

```
failOnRisky, failOnWarning, failOnNotice, failOnDeprecation,
failOnPhpunitDeprecation, failOnPhpunitWarning,
failOnIncomplete, failOnSkipped, failOnEmptyTestSuite,
beStrictAboutOutputDuringTests,
beStrictAboutTestsThatDoNotTestAnything,
beStrictAboutChangesToGlobalState
```

Rules:
- **Never** call `markTestSkipped()`, `markTestIncomplete()`, or `$this->expectDeprecation*()` as a
  way to make a red test green. If the dependency/extension/environment is missing, either (a) mock
  it, (b) make it a required dev-dep, or (c) delete the test. Skipping is not an option.
- **Never** downgrade the fail flags above in any `phpunit.xml*` file. Not even temporarily.
- **Never** write tests that depend on ambient state (network, env vars, filesystem) without a mock.
  Ambient-state tests are the reason we used to see `S` markers — they are banned.
- **Any** PHP warning, notice, or deprecation raised during a test must fail the test. The listed
  attributes enforce this for the PHPUnit process. For HTTP E2E tests against playground servers,
  the playground entry points install a strict error handler that converts warnings/notices into
  `ErrorException`, turning them into HTTP 500s that fail the fixture assertion.
- CI runs the suite **once**, with coverage on the primary matrix cell (ubuntu-latest / PHP 8.4)
  and without coverage on the rest. Never add a second "run tests without coverage" step.

If you see yourself reaching for `markTestSkipped`, stop and ask for help writing a proper mock.

## Hard Timeouts — NEVER RAISE

The test infrastructure is aggressively time-boxed so that nothing — PHPUnit, Vitest, fixtures,
network calls, socket reads — can hang a Claude Code hook or a CI run. These limits are the
**ceiling, not a target**. You are **forbidden** from raising any of them. If a test or command
breaches a limit, fix the slow code (mock I/O, reduce fixture data, split the test, skip a broken
service) — do not edit the ceiling.

| Scope | Limit | Where |
|-------|------:|-------|
| Full test suite (PHPUnit / Vitest) | `180s` | `Makefile` — `TEST_TIMEOUT` |
| Single playground fixture / scenario run | `120s` | `Makefile` — `FIXTURE_TIMEOUT` |
| Helper poll / daemon ping | `15s` | `Makefile` — `HELPER_TIMEOUT` |
| PHPUnit — small / medium / large / default test | `2s / 5s / 10s / 10s` | all `phpunit.xml.dist` |
| PHPUnit — PHP `max_execution_time` / `default_socket_timeout` | `15s / 10s` | all `phpunit.xml.dist` `<ini>` |
| Vitest — `testTimeout` / `hookTimeout` (jsdom) | `10s / 10s` | `libs/frontend/vitest.config.ts` |
| Vitest — `testTimeout` / `hookTimeout` (browser) | `15s / 15s` | `libs/frontend/vitest.browser.config.ts` |
| `FixtureRunner` HTTP timeout | `15s` hard cap | `libs/Testing/src/Runner/FixtureRunner.php` |
| `DebugDataFetcher` retry deadline | `15s` | `libs/Testing/src/Runner/DebugDataFetcher.php` |
| `InspectorClient` HTTP timeout | `15s` hard cap | `libs/McpServer/src/Inspector/InspectorClient.php` |
| `AcpDaemonManager` session-start / prompt | `30s / 30s` hard cap | `libs/API/src/Llm/Acp/AcpDaemonManager.php` |
| `RequestController::request` re-execute client | `15s` (connect 5s) | `libs/API/src/Inspector/Controller/RequestController.php` |
| `FrontendUpdateCommand` GitHub release check | `10s` (connect 5s) | `libs/Cli/src/Command/FrontendUpdateCommand.php` |
| `FrontendUpdateCommand` ZIP download | `30s` (connect 5s) | `libs/Cli/src/Command/FrontendUpdateCommand.php` |

Rules:
- **Never** bump a number in the table above to make something pass. Fix the underlying code.
- **Never** add a new network/socket/subprocess call without an explicit timeout ≤ the matching limit.
- New tests must finish well under `defaultTimeLimit` (10s). If a test legitimately takes longer,
  it belongs in `@group e2e` or `@group playground`, which are excluded from the default suite.
- The `timeout` wrapper in the Makefile (`$(call with_timeout,…)`) is mandatory for any Make target
  that invokes PHPUnit, Vitest, or a playground fixture run.

## Project Overview

ADP (Application Development Panel) is a **framework-agnostic, language-agnostic** debugging and development panel.
It collects runtime data (logs, events, requests, exceptions, database queries, etc.) from applications and provides
a web UI to inspect, analyze, and debug them.

The project is currently a fork/consolidation from Yii Debug into a single monorepo, with the goal of becoming
fully framework-independent. Adapters exist for Yii 3, Symfony, Laravel, Yii 2, and Cycle ORM (database schema only).

## Tech Stack

- **Backend**: PHP 8.4, PSR standards (PSR-3, PSR-7, PSR-11, PSR-14, PSR-15, PSR-16, PSR-17, PSR-18)
- **Frontend**: React 19, TypeScript 6, Vite 8, Material-UI (MUI) 7, Redux Toolkit 2
- **Build**: Composer (PHP), npm workspaces + Lerna (JS), Docker
- **Testing**: PHPUnit 11 (backend), Vitest (frontend)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [app-dev-panel/app-dev-panel](https://github.com/app-dev-panel/app-dev-panel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
