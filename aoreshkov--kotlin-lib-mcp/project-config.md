---
trigger: always_on
description: MCP server (Kotlin Multiplatform) that, on request, downloads the **sources** of a
---

# kotlin-lib-mcp

MCP server (Kotlin Multiplatform) that, on request, downloads the **sources** of a
Maven-published Kotlin/Java library (e.g. `io.ktor:ktor-client-core:3.5.1`) and exposes
structured information about it — public API surface, KDoc, dependencies/metadata, and raw
source + search — to MCP clients (Claude Code, Claude Desktop, …). An optional Compose
Desktop dashboard runs the same server in-process for control, logs, and cache browsing.

## Modules

- `core/` — **KMP library**: domain model + use cases, no MCP and no UI.
  - `commonMain`: model (`LibraryCoordinate`, `ApiSymbol`, `KDoc`, `DependencyNode`, …) and
    interfaces (`MavenSourceFetcher`, `SourceAnalyzer`, `LibraryCache`).
  - `jvmMain`: JVM implementations — `MavenSourceFetcher` (Ktor client + `.module`/`.pom`
    parsing), `ZipExtractor`, `AnalysisApiSourceAnalyzer`, on-disk cache.
- `server/` — **JVM app**: registers MCP tools and runs the transports. Runnable headless.
- `dashboard/` — **Compose Desktop** control panel (optional). Embeds `server`.
- `tools/` — **asset generator**, never shipped: the SEP-973 icon PNGs. Its *output* does ship —
  `server/src/main/resources/icons/` is read by `icons/Icons.kt` and rides inline in every
  `tools/list` — so the glyph geometry in `GenerateIcons.kt` is product source, not repo artwork.
  Nothing depends on the module; `./gradlew build` compiles it only so it cannot rot.
  `./gradlew :tools:generateIcons`.

`server` and `dashboard` both depend on `core`. `server` runs without Compose.

Not a Gradle module: **`plugin/`** — the Claude Code plugin packaging (`.mcp.json` pinning the
GHCR image to its `<major>.<minor>` tag, plus four skills), listed by the repo-root
`.claude-plugin/marketplace.json` so `/plugin marketplace add aoreshkov/kotlin-lib-mcp` works.
Validate with `claude plugin validate ./plugin --strict`; the image pin and the plugin's own
`version` are bumped by the `/release` skill on minor releases.

## Build & run

```
./gradlew build                                   # build all modules
./gradlew test                                    # unit tests
./gradlew :server:run --args="--transport stdio"             # local MCP (default)
./gradlew :server:run --args="--transport http --port 3000"  # Streamable HTTP
./gradlew :dashboard:run                          # Compose Desktop UI
```

**Verify a change:** compile fast with `./gradlew :server:compileKotlin` (pulls in `core`);
run the affected module's tests, e.g. `./gradlew :core:build`. A `Stop` hook runs the fast
compile automatically when Kotlin sources changed (`.claude/hooks/stop-verify.sh`).

## Tech stack (versions live ONLY in `gradle/libs.versions.toml`)

- Kotlin **2.4.x** · MCP `io.modelcontextprotocol:kotlin-sdk-server:0.15.0` · Ktor **3.5.x**
- kotlinx-serialization-json · kotlinx-coroutines
- Source parsing: Kotlin **Analysis API (standalone mode)** — must be version-matched to Kotlin
- Compose Multiplatform (Desktop)
- Logging: Kermit (common) → SLF4J/Logback (JVM sink) · Tests: kotlin-test, coroutines-test, Ktor `MockEngine`

## Conventions & gotchas

- **Versions are centralized** in `gradle/libs.versions.toml`. Kotlin and the Analysis API
  artifacts must share the **exact same version** — always bump them together (use the
  `/analysis-api-bump` skill; cut releases with `/release`). The catalog follows Gradle's
  naming guidance (1–3 dash-separated segments, camelCase inside a segment) and carries **no
  entry a build script doesn't consume** — in particular `[plugins]` holds only the one alias
  a module applies with `alias(...)`; everything else is applied by id from `build-logic`.
- **`core`'s public ABI is locked** by the Kotlin Gradle plugin's built-in `abiValidation`
  (`kmp-library.gradle.kts`; the standalone binary-compatibility-validator it replaces is
  frozen). `./gradlew build` runs `checkKotlinAbi`; intentional API changes need
  `./gradlew updateKotlinAbi` and a committed `core/api/core.api`. The DSL is still
  `@ExperimentalAbiValidation`, so treat a Kotlin bump as able to break it.
- **stdio transport: NEVER write to stdout** except MCP protocol frames. All logging goes to
  stderr or a file, or it corrupts the protocol stream.
- **Core parsing/fetch gotchas** (Analysis API isolation, per-target KMP source jars) live in
  `.claude/rules/analysis-api.md` — loaded automatically when you edit `core/` sources.
- **Cache first.** Downloads + parsed index are cached on disk keyed by
  `group/artifact/version`; tools read the cache. Call `fetch_library` to warm it.

## MCP tools

`fetch_library` · `list_packages` · `list_declarations` · `get_api_signature` · `get_kdoc` ·
`get_source` · `search_source` · `diff_versions` · `get_dependencies` · `list_versions` ·
`get_latest_version`

`fetch_library` also accepts a version-less `group:artifact` or `group:artifact:latest` and
resolves the latest stable release (canonical `<release>`/`<latest>` from `maven-metadata.xml`,
with a semantic-version fallback in `core/util/MavenVersions.kt`).

Cached library indexes are also exposed as MCP **resources** (one static resource per cached
library plus a `kotlinlib://{group}/{artifact}/{version}/index` **resource template**), and an

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aoreshkov/kotlin-lib-mcp](https://github.com/aoreshkov/kotlin-lib-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-18 -->
