---
trigger: always_on
description: This document outlines foundational mandates, architectural patterns, and project-specific conventions to ensure high-quality, idiomatic, and consistent code from the first iteration. When modifying this file, always review the full document to prevent the introduction of duplicate instructions and ensure the content remains coherent and logically organized.
---

# Engineering Standards for Gemini CLI

This document outlines foundational mandates, architectural patterns, and project-specific conventions to ensure high-quality, idiomatic, and consistent code from the first iteration. When modifying this file, always review the full document to prevent the introduction of duplicate instructions and ensure the content remains coherent and logically organized.

## Core Mandates

### 1. Rigorous Import Management
- **Addition:** When adding new symbols, ensure the corresponding import is added.
- **Removal:** When removing the last usage of a class or symbol from a file (e.g., removing a `@Inject Clock clock;` field), **immediately remove the associated import**. Do not wait for a build failure to identify unused imports.
- **No Redundant Qualifications:** NEVER use fully qualified class names (e.g., `java.time.temporal.ChronoUnit.DAYS`) in code when an import can be used instead. Always prefer adding an import and using the simple name.
- **Static Imports for Utilities:** Always statically import methods from utility classes like `DateTimeUtils` or `CacheUtils`. (e.g., use `toInstant(...)` instead of `DateTimeUtils.toInstant(...)`).
- **Checkstyle:** Proactively fix common checkstyle errors (line length > 100, formatting, unused imports) during the initial code write. Do not wait for CI/build failures to address these, as iterative fixes are inefficient.
- **Verification**: Before finalizing any change, scan the imports section for redundancy.
- **License Headers**: When creating new files, ensure the license header uses the current year (e.g., 2026). Existing files should retain their original year.

## 2. Time and Precision Handling

- **UTC Timezones:** Do not use `ZoneId.of("UTC")`. Use a statically imported `UTC` from `ZoneOffset` instead (`import static java.time.ZoneOffset.UTC;`).
- **Clock Injection:**
    - Avoid direct calls to `Instant.now()`, `OffsetDateTime.now()`, or `System.currentTimeMillis()`.
    - Inject `google.registry.util.Clock` (production) or `google.registry.testing.FakeClock` (tests).
    - Use `clock.nowDate()` to get a `LocalDate` in UTC, or `clock.nowDateTime()` to get an `OffsetDateTime` in UTC.
    - When defining timestamps for tests, prefer using a fixed, static constant (e.g., `Instant.parse("2024-03-27T10:15:30.105Z")`) over capturing `clock.now()` to prevent flaky tests caused by the passage of real time.
- **Beam Pipelines:**
    - Ensure `Clock` is serializable (it is by default in this project) when used in Beam `DoFn`s.
    - Pass the `Clock` through the constructor or via Dagger provider methods in the pipeline module.
- **Command-Line Tools:**
    - Use `@Inject Clock clock;` in `Command` implementations.
    - The `clock` field should be **package-private** (no access modifier) to allow manual initialization in corresponding test classes.
    - In test classes (e.g., `UpdateDomainCommandTest`), manually set `command.clock = fakeClock;` in the `@BeforeEach` method.
    - Base test classes like `EppToolCommandTestCase` should handle this assignment for their generic command types where applicable.

### 3. Dependency Injection (Dagger)
- **Concrete Types:** Dagger `inject` methods must use explicit concrete types. Generic `inject(Command)` methods will not work.
- **Test Components:** Use `TestRegistryToolComponent` for command-line tool tests to bridge the gap between `main` and `nonprod/test` source sets.

### 4. Database Consistency
- **Transaction Management:**
    - **Top-Level:** Define database transactions (`tm().transact(...)`) at the highest possible level in the call chain (e.g., in an Action, a Command, or a Flow). This ensures all operations are atomic and handled by the retry logic.
    - **DAO Methods:** Avoid declaring transactions inside low-level DAO methods. Use `tm().assertInTransaction()` to ensure that these methods are only called within a valid transactional context.
    - **Utility/Cache Methods:** Use `tm().reTransact(...)` for utility methods or Caffeine cache loaders that might be invoked from both transactional and non-transactional paths.
        - `reTransact` will join an existing transaction if one is present (acting as a no-op) or start a new one if not.
        - This is particularly useful for in-memory caches where the loader must be able to fetch data regardless of whether the caller is currently in a transaction.
        - **Test Helpers & Timestamps:** If a static test helper method (like in `DatabaseHelper`) needs the database transaction time but might be called from outside a transaction, using `tm().reTransact(tm()::getTxTime)` is acceptable. However, NEVER wrap it redundantly like `tm().transact(() -> tm().reTransact(tm()::getTxTime))`. If you are just setting an arbitrary timestamp in a test where the exact DB transaction time isn't strictly required, prefer `Instant.now()` or `clock.now()` to avoid creating unnecessary database transactions.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [google/nomulus](https://github.com/google/nomulus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
