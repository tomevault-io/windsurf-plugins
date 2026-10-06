---
trigger: always_on
description: This document provides instructions for AI coding agents (GitHub Copilot, Claude Code, and others) to follow when assisting with development in this repository.
---

# Agent Instructions

This document provides instructions for AI coding agents (GitHub Copilot, Claude Code, and others) to follow when assisting with development in this repository.

## Repository Overview

* `apps/` holds the Scala backends (packages under `org.lfdecentralizedtrust.splice`) and React frontends: `validator` (also serves the wallet and ANS backends), `sv`, `scan`, `splitwell`, the shared `common` (`apps-common`; `apps/common/sv` is SV-specific), and `app` (`apps-app`: the release bundle, entry point `SpliceApp.scala`, and the integration tests).
* Stores ingest the ledger update stream asynchronously. Automation runs as `*Trigger` classes that react to ledger state and submit follow-up commands.
* `daml/` holds the Daml packages, one sbt project each. `token-standard/` defines the CIP-0056 token standard APIs, which `daml/splice-amulet` implements.
* `canton/` is a copy of open-source Canton, not Splice code. Avoid editing it; if you must, record the change in `CANTON_CODE_CHANGES.md` (see "Bumping Our Canton fork" in [MAINTENANCE.md](MAINTENANCE.md)).
* `cluster/` holds the Helm charts, Pulumi deployment code, and Docker Compose setup.

## Testing

Every contribution must be tested in an automated test. For further details see the [Testing README](TESTING.md).

## Building and Running Tests

* Run `sbt` inside the direnv/nix dev shell, e.g. `direnv exec . sbt -batch "apps-scan/testOnly <FullyQualifiedSuite>"`. `-batch` avoids the interactive prompt; redirect output to a file and grep for `^\[error\]` and `Tests:`.
* Test code compiles with `-Xfatal-warnings` and `-Wunused:privates`/`-Wunused:params`, so an unused private member or parameter fails `Test/compile`.
* To format specific files: `sbt "scalafmtOnly <files...>"`.
* Integration tests under `apps/app/src/test` need a running Canton from `./start-canton.sh`, including suites based on `IntegrationTestWithIsolatedEnvironment`. Without it they fail with `java.io.FileNotFoundException: canton.tokens`.
  * Start only what the test needs: `./start-canton.sh -d -w` for wallclock tests, `-s` for simtime tests. Add `-p local` to use a local Postgres instead of Docker.
  * `start-canton.sh` manages Postgres itself; do not start it separately with `scripts/postgres.sh`. Stop with `./stop-canton.sh` (pass the same Postgres mode, e.g. `./stop-canton.sh local`).
  * Restart Canton (`./stop-canton.sh` then `./start-canton.sh`) whenever `nix/canton-sources.json` changes, e.g. after a checkout or rebase: it pins the Canton version that `start-canton.sh` runs.
  * Run a single test with `sbt "apps-app/testOnly <FullyQualifiedSuite> -- -z \"<test name>\""`.
  * Suites named `*FrontendIntegrationTest` drive the frontends through Selenium and also need `./start-frontends.sh`.
* Do not run bare `sbt test`; use `testOnly` and leave the full suite to CI.
* CI runs `sbt checkErrors` after tests, which fails on WARN/ERROR lines in the logs under `log/` that don't match `project/ignore-patterns/`. The logs are not rotated, so stale errors from earlier runs also show up; clear `log/` before a run you want to check. The main logs are `log/canton_network_test.clog` (test run and Splice apps) and `log/canton.clog` / `log/canton-simtime.clog` (Canton).
* `sbt formatFix` applies scalafmt, scalafix, npm fixes and copyright headers (`make format` additionally formats cluster code). `sbt lint` runs the static checks; note that its `scalafixAll` step can rewrite files.
* If dependency resolution fails with a 404 for a Canton snapshot artifact (e.g. `com.daml:bindings-java:<version>-snapshot...`), the pinned snapshot has likely expired; rebase onto the latest `main` and reload direnv.

## Writing Integration Tests

* Stores ingest asynchronously: never assert on store state immediately after an action. Use `actAndCheck`, `eventually` or `eventuallySucceeds`.
* All test cases of a suite extending `IntegrationTest` share one environment (`IntegrationTestWithIsolatedEnvironment` creates one per test case). Use `perTestCaseName(...)` for party and other names to avoid collisions between test cases.
* Wrap actions that are expected to log warnings or errors in `loggerFactory.assertLogs` (or a related `SuppressingLogger` method); otherwise `checkErrors` fails in CI.
* Simulated-time tests mix in `TimeTestUtil` and move the clock with `advanceTime` and related helpers.

## DB Migrations

Refer to [the main README on migrations](apps/common/src/main/resources/db/migration/README.md). Most importantly, never modify a deployed migration script, not even its comments; add a new migration instead.

## Daml Changes

### Daml Lock Files

If you make intentional changes in Daml code, run `sbt damlDarsLockFileUpdate` and commit the updated `dars.lock` file along with your dar changes.

### Backwards-compatible Daml changes

All Daml changes must be backwards-compatible. See the [Smart Contract Upgrading Reference from the Canton docs](https://docs.canton.network/appdev/deep-dives/smart-contract-upgrading-reference).

When adding to enums, make sure to only add further nullary constructors to types that only have nullary constructors.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [canton-network/splice](https://github.com/canton-network/splice) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
