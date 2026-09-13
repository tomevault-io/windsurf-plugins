---
trigger: always_on
description: This Kotlin Multiplatform SDK requires careful navigation of cross-platform concurrency, wire-protocol semantics, and architectural boundaries. Follow these guidelines to work effectively.
---

# Copilot Instructions for meshtastic-sdk

This Kotlin Multiplatform SDK requires careful navigation of cross-platform concurrency, wire-protocol semantics, and architectural boundaries. Follow these guidelines to work effectively.

## Prerequisites

**Environment setup is critical.** Before any work:
- JDK 21 (verify with `java -version`)
- Submodules initialized: `git clone --recurse-submodules` or `git submodule update --init --recursive`
  - If `proto/src/protobufs/` is empty or stale, protocol/generated code will be wrong
- Android SDK API 35 + `ANDROID_HOME` (for Android targets)
- Xcode 15+ (macOS/iOS targets)
- Gradle 8.4+ (bundled; no separate install needed)

## Build, Test, Lint

**Full gate (run before any PR):**

```bash
./gradlew spotlessApply check                    # format + unit tests + lint + architecture rules + ABI check
bash .github/tooling/check.sh                    # schema + markdown frontmatter validation
```

**Targeted workflows (faster iteration):**

```bash
./gradlew detekt spotlessCheck                   # lint only, no build
./gradlew jvmTest                                # JVM unit tests only
./gradlew iosSimulatorArm64Test                  # iOS unit tests (macOS only)
./gradlew :core:verifyModuleBoundary             # enforce architecture rules
./gradlew checkKotlinAbi                         # detect public API changes
```

**ABI baseline workflow (intentional public API changes only):**

```bash
./gradlew updateKotlinAbi                        # refresh api/*.api baseline files
git add api/                                     # commit the updated baselines
git commit -s -m "chore: ABI baseline refresh"
```

Omit this if your change is internal (not public API). `checkKotlinAbi` will fail with guidance if needed.

**Test coverage:**
- Unit tests in `src/{jvm,android,ios,common}Test/kotlin/`
- Platform-specific tests respect module boundaries; no transport/storage logic in `:core:test`
- `gradle spotlessApply` auto-fixes formatting; check results before committing

## Architecture & Module Boundaries

The SDK enforces hard architectural rules via detekt `ForbiddenImport` + `:core:verifyModuleBoundary` Gradle task.

**Dependency graph (strict):**
- `:core` → `:proto` only (no transports, no storage)
- `:transport-ble`, `:transport-tcp`, `:transport-serial` → `:core` + their own protocol impls
- `:storage-sqldelight` → `:core` (no transport coupling)
- Each transport/storage is independently pluggable

**Hard rules:**
- Do not add transport/storage implementation code to `:core`
- Do not add `kotlin.Result<T>` to public API (see ADR-005 for response-shape rules)
- Engine concurrency model is single-writer actor; no mutex/atomic/synchronized in hot paths (see ADR-002)
- Public API must follow shape policy: throw exceptions, AdminResult wrappers, or Flows — never Result

**Where to look:**
- Module graph: [`docs/architecture/module-graph.md`](docs/architecture/module-graph.md)
- Architecture decisions: [`docs/decisions/002-architecture.md`](docs/decisions/002-architecture.md)
- Enforcement matrix: [`docs/architecture/enforcement.md`](docs/architecture/enforcement.md)
- ADR index: [`docs/decisions/`](docs/decisions/)

## Multiplatform Concurrency & Dispatchers

**Dispatcher shadowing quirk (KMP gotcha):**
On Native (iOS), `Dispatchers.IO` from common code fails with "it is internal in kotlinx.coroutines.Dispatchers". The public extension property is shadowed by an internal member. Workaround:

```kotlin
// ✗ WRONG: will fail on Native
Dispatchers.IO

// ✓ RIGHT: use per-platform expect/actual
expect val defaultStorageDispatcher: CoroutineDispatcher

// Apple actual:
internal actual val defaultStorageDispatcher: CoroutineDispatcher =
    Dispatchers.Default.limitedParallelism(4, "sqldelight-storage")
```

See `storage-sqldelight/src/{jvm,android,apple}Main/.../StorageDispatcher.*.kt` for reference.

**Main-safety of suspend operations:**
All suspend operations in `:core` and storage are safe to call from UI/main thread (they dispatch off-thread). Verify MutableSharedFlow emissions in `MeshEngine`.

**Thread-local JDBC quirk:**
`JdbcSqliteDriver(url)` uses ThreadLocal connection pools. Post-constructor `driver.execute("PRAGMA …")` only affects the creating thread. Always apply PRAGMAs via JDBC URL properties or `AndroidSqliteDriver.Callback.onConfigure()`.

## Public API & Versioning

**Before committing public API changes:**
1. Read [`docs/decisions/005-api-shape.md`](docs/decisions/005-api-shape.md) — documents response-shape policy
2. Run `./gradlew checkKotlinAbi` — detects unintended ABI drift
3. If changes are intentional, run `./gradlew updateKotlinAbi` and commit `api/*.api` files
4. Verify no `kotlin.Result<T>` sneaked into public signatures

**Versioning:**
- Track [`docs/versioning.md`](docs/versioning.md) for SemVer + pre-1.0 compatibility guarantees
- Breaking changes to public API are allowed pre-1.0 but require `updateKotlinAbi`

## Wire Protocol & Storage

**Protocol behavior:**
- Handshake, NodeDB, ACK correlation, retries: [`docs/protocol.md`](docs/protocol.md)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [meshtastic/meshtastic-sdk](https://github.com/meshtastic/meshtastic-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-13 -->
