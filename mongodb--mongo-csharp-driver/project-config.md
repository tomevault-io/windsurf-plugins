---
trigger: always_on
description: JSON-driven runners for the cross-driver test suite. The JSON test files mirror those at `https://github.com/mongodb/specifications` and are embedded as project resources; the C# code in this directory wires them up to xUnit.
---


# Specifications — AGENTS.md

JSON-driven runners for the cross-driver test suite. The JSON test files mirror those at `https://github.com/mongodb/specifications` and are embedded as project resources; the C# code in this directory wires them up to xUnit.

## Two formats

- **Legacy v1** — older style. Three things to know:
  - **Base-class variance.** Some legacy runners extend `LoggableTestClass` (e.g. CRUD, InWindow, `connection-string/ConnectionStringTestRunner`); others are plain xUnit `[Theory]` classes with no shared base (e.g. `AuthTestRunner`, `ServerSelectionTestRunner`, `BsonCorpusTestRunner`, the `read-write-concern/ConnectionStringTestRunner`). Note the asymmetry across the two `ConnectionStringTestRunner` files: the one under `connection-string/` extends `LoggableTestClass`, the one under `read-write-concern/` does not. Don't assume a shared base — check the file. All of them pair with a custom `JsonDrivenTestCaseFactory` for resource discovery.
  - **`MongoClientJsonDrivenTestRunnerBase`** is a heavier shared base; in this repo only `ClientSideEncryptionTestRunner` derives from it. It adds resource matching, `runOn`/`minServerVersion`/`maxServerVersion` filtering, EventCapturer setup, FailPoint cleanup, lifecycle. Not used by other legacy runners.
  - **Path-prefix property naming is inconsistent across runners.** Most legacy runners override a singular `PathPrefix` property (e.g. `AuthTestRunner`, `BsonCorpusTestRunner`, `ClientSideEncryptionTestRunner`, the SDAM / CMAP runners), while a few override the plural `PathPrefixes` (e.g. `CrudTestRunner`, `server-selection/ServerSelectionTestRunner`, `server-selection/InWindowTestRunner`, `connection-string/ConnectionStringTestRunner`, the `client-side-encryption/prose-tests/ClientEncryptionProseTests`). Check the runner's existing override before adding a new resource namespace — guessing the wrong name yields zero tests discovered.
  - **Resource-suffix variance.** Spec-specific: CRUD uses `MongoDB.Driver.Tests.Specifications.crud.tests.v1` (with the `.v1` segment), while many other legacy runners use just `…tests` with no `.v1` segment.
- **Unified Test Format (UTF)** — newer; preferred for new specs. Discovered via `[UnifiedTestsTheory(...)]` methods on the `UnifiedTestSpecRunner` class (one class with many theory methods, not one class per spec). The `UnifiedTestsDiscoverer` reads UTF JSON files from embedded-resource namespaces and instantiates xUnit cases.
  - Resource-namespace gotcha: upstream spec resource paths use hyphens (`change-streams/`, `retryable-reads/`, `client-side-operations-timeout/`); not every hyphenated form has a matching on-disk folder under `Specifications/` (e.g. there is no `client-side-operations-timeout/` directory, the embedded-resource namespace exists nonetheless). Because C# resource names follow identifier rules, hyphens become underscores in the namespace — e.g. `change_streams.tests.unified`, `retryable_reads.tests.unified`, `client_side_operations_timeout.tests`. A handful also drop the `.unified` suffix entirely (`sessions.tests`, `versioned_api.tests`). Cross-check the `[UnifiedTestsTheory(...)]` argument against the actual embedded-resource list when you add a spec.

Many spec areas have **both** legacy and unified subdirectories during the migration window; new tests should land in unified.

## Layout

- `Runner/` — contains only `MongoClientJsonDrivenTestRunnerBase`. Handles schema validation, `runOn`/`minServerVersion`/`maxServerVersion` filtering, EventCapturer setup, FailPoint cleanup, lifecycle. Its resource-matching property is the singular `PathPrefix` (overridden by `ClientSideEncryptionTestRunner`).
- `tests/MongoDB.Driver.Tests/UnifiedTestOperations/` (sibling of this directory, **not** a subdirectory) — ~100+ operation classes implementing one of `IUnifiedTestOperation`, `IUnifiedEntityTestOperation`, `IUnifiedSpecialTestOperation`, or `IUnifiedOperationWithCreateAndRunOperationCallback` (e.g., `UnifiedAggregateOperation`, `UnifiedClientBulkWriteOperation`, `UnifiedAssertEventCountOperation`). New operations land there with matchers in the `Matchers/` subdirectory.
- One subdirectory per spec area. The on-disk list grows over time and is the authoritative source — `ls tests/MongoDB.Driver.Tests/Specifications/` is the only reliable enumeration. A few representative entries (not exhaustive): `crud/`, `server-discovery-and-monitoring/`, `connection-monitoring-and-pooling/`, `client-side-encryption/`, `change-streams/`, `auth/`, plus the prose-only folders `transactions/`, `sessions/`, `retryable-reads/`, `retryable-writes/` (called out separately in the next bullet).
  - **Prose-only C# folders.** For `transactions/`, `sessions/`, `retryable-reads/`, and `retryable-writes/`, the C# classes in those folders are prose-only — they hold only spec-prose tests written as ordinary xUnit, not a JSON runner class.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mongodb/mongo-csharp-driver](https://github.com/mongodb/mongo-csharp-driver) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
