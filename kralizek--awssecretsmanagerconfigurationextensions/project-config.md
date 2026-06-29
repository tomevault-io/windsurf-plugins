---
trigger: always_on
description: This file is the operational reference for automated coding agents working in this repository.
---

# AGENTS.md — Coding-agent guide for Kralizek.Extensions.Configuration.AWSSecretsManager

This file is the operational reference for automated coding agents working in this repository.
It is not a replacement for [`README.md`](README.md) or [`CONTRIBUTING.md`](CONTRIBUTING.md) — read those first for background.

---

## 1. Repository purpose

**Kralizek.Extensions.Configuration.AWSSecretsManager** is a .NET configuration provider that integrates AWS Secrets Manager with `Microsoft.Extensions.Configuration`. The library offers three explicit loading modes:

- **Discovery**: List all secrets and batch-fetch their values (for zero-config, all-secrets scenarios)
- **KnownSecrets**: Batch-fetch a fixed set of known secrets (for multi-secret scenarios)
- **KnownSecret**: Fetch exactly one secret (for single-secret scenarios)

The library is designed for high-performance scenarios with built-in telemetry, structured logging, and minimal API calls.

See [`README.md`](README.md) for usage examples and [`MIGRATION.md`](MIGRATION.md) for breaking change history.

---

## 2. Repository layout

```
src/
  Kralizek.Extensions.Configuration.AWSSecretsManager/
    ├── SecretsManagerExtensions.cs          ← Entry point; contains AddSecretsManager* methods
    ├── SecretsManagerDiscoveryOptions.cs    ← Discovery mode configuration
    ├── SecretsManagerKnownSecrets*.cs       ← KnownSecrets mode configuration
    ├── SecretValueContext.cs                ← Shared secret context
    ├── SecretsManagerLogging.cs             ← Structured logging definitions
    ├── SecretsManagerTelemetry.cs           ← Telemetry / OpenTelemetry integration
    ├── DuplicateKeyHandling.cs              ← Enum for duplicate key strategies
    ├── MissingSecretValueException.cs       ← Custom exception
    └── Internal/                            ← Implementation details (not part of public API)

tests/
  Tests.Extensions.Configuration.AWSSecretsManager/
    ├── SecretsManagerExtensionsTests.cs     ← Integration tests for all modes
    ├── ConfigurationProviderExtensions.cs   ← Test utility extensions
    └── Internal/                            ← Internal test fixtures
    └── Types/                               ← Test data types

samples/
  Sample1/ through Sample8/                  ← Individual mode demonstrations
  SampleWeb/                                 ← ASP.NET Core integration example

docs/
  (If present) Architecture, design decisions, limitations
```

---

## 3. Before making changes

Depending on the task, inspect the relevant files before touching any code:

| Task type | Files to read first |
|-----------|---------------------|
| New loading mode | `README.md` (Limitations section), `SecretsManagerExtensions.cs`, `tests/` |
| Add Discovery option | `SecretsManagerDiscoveryOptions.cs`, `README.md` §Discovery |
| Add KnownSecrets option | `SecretsManagerKnownSecrets*.cs`, `README.md` §KnownSecrets |
| Add KnownSecret option | `SecretsManagerExtensions.cs`, `README.md` §KnownSecret |
| Error handling / exception | `MissingSecretValueException.cs`, `SecretsManagerLogEvents.cs` |
| Telemetry / observability | `SecretsManagerTelemetry.cs`, `SecretsManagerLogging.cs` |
| Breaking change | `MIGRATION.md`, `README.md` |
| Sample/documentation | `samples/`, `README.md` |

---

## 4. Working rules

- **Keep PRs small and focused** — one logical change per PR
- **No unrelated refactors** — fix only what the task requires
- **Test all three modes** — if a change affects the core API or configuration, ensure it works across Discovery, KnownSecrets, and KnownSecret
- **Preserve backwards compatibility** unless the task explicitly requires a breaking change (document in `MIGRATION.md` and update version)
- **Extend existing patterns** (e.g., new option in existing `*Options.cs` class) rather than inventing new ones
- **Update docs and tests when behaviour changes**:
  - If public API changes, update `README.md` and XML docs
  - If logging/telemetry changes, update `SecretsManagerLogging.cs` or `SecretsManagerTelemetry.cs`
  - If a mode's behaviour changes, update the corresponding sample
  - If a breaking change, update `MIGRATION.md`
- **AWS API efficiency** — minimize redundant AWS Secrets Manager calls; batch operations where possible

---

## 5. Required validation before finishing

Run all checks from the repository root before declaring work done:

```bash
# 1. Format validation
dotnet format --verify-no-changes

# 2. Build — warnings are treated as errors
dotnet build --no-incremental -warnaserror

# 3. All tests
dotnet test

# 4. Check code analysis (optional, if enabled)
dotnet analyze
```

For changes affecting a specific mode, also run the corresponding sample:

```bash
# For Discovery mode changes
cd samples/Sample1
dotnet run

# For KnownSecrets mode changes
cd samples/Sample2
dotnet run

# For KnownSecret mode changes
cd samples/Sample3
dotnet run

# For web integration
cd samples/SampleWeb
dotnet run
```

---

## 6. Task-specific guidance

### Configuration changes (new option, new mode)

- Add the option class or extend an existing `*Options.cs` file
- Add the extension method to `SecretsManagerExtensions.cs`
- Add integration tests to `SecretsManagerExtensionsTests.cs`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kralizek/AWSSecretsManagerConfigurationExtensions](https://github.com/Kralizek/AWSSecretsManagerConfigurationExtensions) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
