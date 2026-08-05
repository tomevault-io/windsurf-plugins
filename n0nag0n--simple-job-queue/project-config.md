---
trigger: always_on
description: This file provides specific instructions for AI coding agents (and human contributors) working on the project. It is based on the original repo audit and the approved Phase 1 + Phase 2 plans.
---

# AGENTS.md — Coding Guidelines for simple-job-queue

This file provides specific instructions for AI coding agents (and human contributors) working on the project. It is based on the original repo audit and the approved Phase 1 + Phase 2 plans.

## Non-Negotiable Rules

### 1. API Stability (Highest Priority)
- **Maintain the exact same public API at all times.**
- Never change signatures, visibility, return types, or documented/observable behavior of existing public methods:
  - `__construct`, `setOptions`/`getOptions`, `setPipeline`/`getPipeline`
  - `getCache`/`flushCache`, `addQueueConnection`
  - `selectPipeline`, `watchPipeline`
  - `addJob`, `getNextJobAndReserve`, `getNextBuriedJob`
  - `deleteJob`, `buryJob`, `kickJob`
  - `getJobId`, `getJobPayload`
  - The four `is*QueueType()` methods
- Job representations must stay consistent: arrays for DB backends (`['id' => ..., 'payload' => ...]`), Pheanstalk Job objects for beanstalkd (accessed via the getter methods).
- **New functionality must be additive only** (new methods like `releaseJob()`, new config options with safe defaults, new count helpers).

### 2. No Breaking Changes for Existing Users
- **Critical for live systems**: Many users have long-running MySQL (or Postgres/SQLite) queues with existing tables and historical job data. Nothing we do can break those setups, their data, or the queries that operate on them.
- All defaults must preserve current behavior (e.g., the 5-minute stale reserved timeout must remain the default).
- Changes must not alter error handling behavior that callers may rely on (many SQL paths currently swallow + `error_log`; tests document this).
- **Schema rule (absolute)**: Only ever use `CREATE TABLE IF NOT EXISTS`. Never emit `ALTER TABLE`, `DROP TABLE` (in library code), or any migration. Any schema tweaks (e.g., column nullability) live exclusively inside the creation block and only affect tables that do not yet exist. Existing tables and their data are sacred.
- All INSERT/UPDATE/SELECT must continue to work on rows created by older versions of the library (use the same columns and logic that have always been present).

Example safe pattern (from current code):
```php
$this->connection->exec("CREATE TABLE IF NOT EXISTS {$table_name} ( ... )");
```
No ALTERs. Period.

### 3. PHP 7.4+ Compatibility (Hard Requirement)
- The project **requires PHP ^7.4 || ^8.0** (see `composer.json` and CI matrix).
- CI explicitly tests on PHP 7.4, 8.1, and 8.3.
- **Dev dependency upgrades are constrained**: PHPUnit must stay within the 9.x series (latest ^9.6). PHPUnit 10+ drops support for PHP < 8.1.
- Explicitly state the PHP 7.4 minimum in:
  - `composer.json` (already present)
  - README.md (install/requirements section)
  - This AGENTS.md
- Never propose or make changes that would drop PHP 7.4 support without a major version bump discussion.

### 4. Always Use the Safety Net (New CI)
- Run these commands before considering any change complete:
  - `composer validate --strict`
  - `composer test` (or `vendor/bin/phpunit --testdox`)
  - `composer test-coverage` (when Xdebug is available; must not regress coverage)
- All work must pass the GitHub Actions CI workflow (`.github/workflows/ci.yml`), including the PHP 7.4 job.
- Use the CI as the primary verification tool for dependency changes and cross-PHP compatibility.

## Preferred Development Patterns

- **Reuse existing internal helpers** (see `src/Job_Queue.php`):
  - `runPreChecks()`
  - `getSqlTableName()` / `getRawTableName()`
  - The four `is*QueueType()` methods
  - `setOptions()` / `getOptions()` (the options system is the approved extension point)
  - Queue-type switch statements for adapter-specific logic (DB vs beanstalkd)
- Follow the established transaction + `FOR UPDATE` (or empty for SQLite) pattern for DB operations that need atomicity.
- For new public methods that require a pipeline/connection, call `runPreChecks()` first.
- Error handling: Match the style of similar methods. Document swallowing behavior in tests if you add new paths that catch exceptions.
- Tests: Use the existing patterns (SQLite `:memory:`, heavy use of mocks for beanstalkd and error cases, the `getProtectedProperty()` reflection helper for testing protected methods). Improve `tearDown()` to handle custom table names.

## Project History & Context (Must Read)
- This library started as a simple DB job queue with optional Beanstalkd support via an adapter pattern.
- Phase 1 (merged PR #10) delivered:
  - Documentation fixes (README examples now valid, comments corrected)
  - CI workflow for sanity (PHP matrix + validate + tests)
  - Non-breaking fixes for multi-`table_name` support, table existence checks, and Postgres PRIMARY KEY in DDL
- The user (maintainer) is a **strong proponent of API stability**. Explicit quote: "I am a HUGE proponent of maintaining the same api in a given library. If things can be fixed but it doesn't break existing functionality, that's what I would prefer."
- Remaining work (this Phase 2 plan) focuses on:
  - Safe dependency hygiene (within PHP 7.4 constraints)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [n0nag0n/simple-job-queue](https://github.com/n0nag0n/simple-job-queue) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
