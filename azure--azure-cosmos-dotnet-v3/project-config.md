---
trigger: always_on
description: <!-- .github/copilot-instructions.md -->
---

<!-- .github/copilot-instructions.md -->
# Copilot / AI assistant instructions — Microsoft.Azure.Cosmos

Purpose: quick, actionable context so an AI coding assistant can be immediately productive in this repo.

- **Big picture**: This repository implements the v3 .NET SDK for Azure Cosmos DB. Major components:
  - `Microsoft.Azure.Cosmos/` — core SDK client, most production code.
  - `Microsoft.Azure.Cosmos.Encryption/` and `Microsoft.Azure.Cosmos.Encryption.Custom/` — client-side encryption extensions.
  - `Microsoft.Azure.Cosmos.Samples/` — runnable examples and usage patterns.
  - `docs/`, `templates/`, and top-level Azure Pipelines YAMLs — CI, packaging and emulator setup.

- **Why this structure**: the SDK core is separated from optional features (encryption, fault-injection, direct mode) so consumers can opt into smaller packages. Versioning and feature flags are centralized in `Directory.Build.props`.

- **Build & test (most common workflows)**:
  - Build solution: `dotnet build Microsoft.Azure.Cosmos.sln -c Release` (or simply `dotnet build`).
  - Run unit/integration tests: `dotnet test --no-build` in the solution or specific test project folders under `**/tests/`.
  - CI uses the YAML files in the repository root and `templates/` — see `templates/emulator-setup.yml` for the Windows emulator script used in CI.

- **Local emulator and integration testing**:
  - The codebase expects the Windows Cosmos DB Emulator in many integration tests. CI installs/starts it via `templates/emulator-setup.yml` (PowerShell scripts that download and launch the MSI and call `Start-Process CosmosDB.Emulator.exe`).
  - If running tests locally on Windows, install the emulator and ensure exclusions and local state paths match what's in `templates/emulator-setup.yml`.

- **Versioning & build flags**:
  - `Directory.Build.props` (repo root and project-level overrides) contains the canonical package versions and MSBuild flags (e.g. `<ClientOfficialVersion>`, `<LangVersion>`, and `DefineConstants` that add `PREVIEW`/`ENCRYPTIONPREVIEW`).
  - Feature/preview builds are gated by MSBuild properties like `IsPreview` or `IsNightly`; set these via `dotnet msbuild /p:IsPreview=true` when needed.

- **Conventions & patterns** (project-specific)
  - Avoid introducing new global build properties; add versions to `Directory.Build.props` where applicable.
  - Tests use the emulator or mocks; integration tests that depend on emulator are usually under `tests/` and expect environment-based setup. Look for CI templates for exact start-up sequence.
  - Strong-name signing keys exist at repo root (`35MSSharedLib1024.snk`, `testkey.snk`); builds may require signing configuration on CI.

- **Integration points & external deps**:
  - Azure Cosmos DB Emulator (Windows) — required for many integration tests.
  - NuGet packaging and pipeline tooling — see `templates/nuget-pack.yml` and the many `azure-pipelines-*.yml` files for packaging/release behavior.

- **Where to look for examples** (use these as source-of-truth snippets):
  - `Directory.Build.props` — versioning and define-constants
  - `templates/emulator-setup.yml` — exact emulator install/start PowerShell used in CI
  - `Microsoft.Azure.Cosmos/` — core SDK patterns (public APIs, partitioning, feed iterator usage)
  - `Microsoft.Azure.Cosmos.Samples/` — minimal runnable samples for usage patterns

- **How AI should produce code/changes here**:
  - **🚫 HARD RULE: NEVER push directly to `main` — NO EXCEPTIONS.** Always create a feature branch and submit a pull request. This rule cannot be overridden.
  - **Branch naming**: Always use the format `users/<user-name>/feature-name` (e.g. `users/ntripician/fix-retry-logic`). Do not use other conventions like `feature/`, `fix/`, or `dev/`.
  - **PR title format**: All pull request titles **must** match the CI lint regex: `(\[Internal\]|\[v4\] )?.{3}.+: (Adds|Fixes|Refactors|Removes) .{3}.+`. The format is `[Optional Prefix] Category: Verb Description` where the verb is one of `Adds`, `Fixes`, `Refactors`, or `Removes`. Optional prefixes are `[Internal]` (for PRs with no customer impact) or `[v4]` (for v4-specific changes). Examples:
    - `Diagnostics: Adds GetElapsedClientLatency to CosmosDiagnostics`
    - `PartitionKey: Fixes null reference when using default(PartitionKey)`
    - `[v4] Client Encryption: Refactors code to external project`
    - `[Internal] Query: Adds code generator for CosmosNumbers for easy additions in the future`
  - Keep changes minimal and focused; prefer small, targeted edits and follow existing code style.
  - **Changelog entry required**: Every PR that modifies the shipped package
    source (anything under `Microsoft.Azure.Cosmos/src/**`, equivalent paths
    for FaultInjection / Encryption packages) must also add a bullet under
    `### Unreleased` in `changelog.md`, in one of the four subsections
    `Features Added` / `Breaking Changes` / `Bugs Fixed` / `Other Changes`.
    Write the entry in **customer-facing language** — not the PR title
    verbatim. If the change has zero customer-observable impact (test-only,
    doc-only, CI-only, purely internal refactor), check the "No changelog
    entry required" box in the PR template and justify briefly. If a
    preview-only change might affect default-config semantics for any

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Azure/azure-cosmos-dotnet-v3](https://github.com/Azure/azure-cosmos-dotnet-v3) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
