---
trigger: always_on
description: **Analysis Date:** 2026-05-13
---

# Coding Conventions

**Analysis Date:** 2026-05-13

## Language Rules (Non-Negotiable)

- All code, comments, identifiers, and KDoc must be in English (AGENTS.md, line 4-5)
- Kotlin + Gradle Kotlin DSL only — no Java files in `src/`
- Target: JVM 21 (`java.toolchain.languageVersion = 21`, compiler option `JVM_21`)
- JSR-305 strict null checking: `-Xjsr305=strict` in `compilerOptions`

## Build Tooling

The Gradle wrapper is pinned to `8.12.1` (see `gradle/wrapper/gradle-wrapper.properties`), which only supports JDKs 8-23 as the launcher. Foojay toolchain auto-provisioning covers compile/test (JDK 21) but not the launcher itself, so `./gradlew` fails when the shell's default `java` is JDK ≥24 (e.g. Homebrew's `openjdk` is now 25).

Invoke gradle with JDK 21 as the launcher:

```bash
JAVA_HOME=$(/usr/libexec/java_home -v 21) ./gradlew <task>
```

Or rely on `.tool-versions` (`java temurin-21.0.10`) via mise/asdf if its shell hook is active. Claude Code's Bash sessions get `JAVA_HOME` pre-set via `.claude/settings.local.json`'s `env` block (gitignored, per-machine).

## Naming Patterns

**Files:**
- One top-level class or object per file, filename matches class name: `AgentSettings.kt`, `PassiveAiScanner.kt`
- Test files: `{SubjectClass}Test.kt` — co-located under `src/test/kotlin/` mirroring the production package

**Classes / Objects:**
- PascalCase for classes, objects, and interfaces: `BackendRegistry`, `HttpBackendSupport`, `AiBackend`
- `object` for stateless singletons / utility namespaces: `Redaction`, `ScannerIssueSupport`, `HttpBackendSupport`, `TestSettings`
- `companion object` for constants and factory members: `RedactionPolicy.fromMode()`, `HealthCheckResult.Healthy`

**Functions:**
- camelCase: `migrateIfNeeded()`, `buildCliHistory()`, `sharedClient()`
- Boolean predicates: `is` prefix — `isAlive()`, `isAvailable()`, `isWindows()`, `isRetryableConnectionError()`
- Factory functions named after return type or `create()`: `PerplexityBackendFactory.create()`, `buildClient()`

**Constants:**
- `SCREAMING_SNAKE_CASE` inside `companion object` or top-level: `CIRCUIT_FAILURE_THRESHOLD`, `CURRENT_SETTINGS_SCHEMA_VERSION`, `DEFAULT_BASE_URL`
- Numeric literals: underscore grouping for readability — `5_000`, `262_144`, `10 * 60 * 1000`

**Variables / Properties:**
- camelCase throughout: `lastUsedAt`, `availabilityLogged`, `cachedSettings`
- `@Volatile` on fields read across threads without synchronized blocks: `systemPromptEntry`, `endpointDedupMinutes`
- `AtomicBoolean` / `AtomicInteger` / `AtomicReference` / `AtomicLong` for lock-free shared state — do not use `synchronized` on primitives

## Kotlin Idioms in Use

**Data classes** — all value objects use `data class` (immutability via `val`, structural equality, free `copy()`):
- `AgentSettings` (`src/main/kotlin/com/six2dez/burp/aiagent/config/AgentSettings.kt`)
- `BackendLaunchConfig`, `ChatMessage`, `TokenUsage` (`src/main/kotlin/com/six2dez/burp/aiagent/backends/BackendTypes.kt`)
- `PassiveAiFinding`, `PassiveAiScannerStatus` (`src/main/kotlin/com/six2dez/burp/aiagent/scanner/PassiveAiScanner.kt`)
- `RedactionPolicy` (`src/main/kotlin/com/six2dez/burp/aiagent/redact/Redaction.kt`)

**Sealed classes** — use for exhaustive sum types where `when` must cover all cases:
```kotlin
// src/main/kotlin/com/six2dez/burp/aiagent/backends/BackendTypes.kt
sealed class HealthCheckResult {
    data object Healthy : HealthCheckResult()
    data class Degraded(val message: String) : HealthCheckResult()
    data class Unavailable(val message: String) : HealthCheckResult()
    data object Unknown : HealthCheckResult()
}
```
- Exhaustive `when` is enforced by the compiler — no `else` branch needed or desired.

**Enums with companion factory** — use `fromString` pattern with `ignoreCase = true` fallback:
```kotlin
enum class PrivacyMode {
    STRICT, BALANCED, OFF;
    companion object {
        fun fromString(raw: String?): PrivacyMode =
            entries.firstOrNull { it.name.equals(raw, ignoreCase = true) } ?: BALANCED
    }
}
```
All enums follow this pattern: `PrivacyMode`, `SeverityLevel`, `PayloadRisk`, `ScanMode`.

**Null safety** — ADR-1 explicitly calls this out:
- Prefer `?: ""` and `?: emptyList()` over `!!` — never use `!!` in production code
- Use `orEmpty()`, `?.trim()`, `.ifBlank { default }` chains: `prefs.getString(KEY).orEmpty().trim().ifBlank { defaultValue() }`
- Montoya API methods that return nullable: always chain safe call + Elvis

**Extension functions** — used for cohesion:
```kotlin
// AgentSettings.kt
fun AgentSettings.toPreprocessorSettings() = ResponsePreprocessorSettings(...)
```

**Coroutines** — used only in the MCP layer:
- `src/main/kotlin/com/six2dez/burp/aiagent/mcp/McpStdioBridge.kt`: `CoroutineScope(Dispatchers.IO + SupervisorJob())`, `runBlocking`
- Ktor server internals (SSE, routing) are inherently coroutine-based via `KtorMcpServerManager`
- Everywhere else concurrency is handled with `java.util.concurrent` primitives (`Executors`, `AtomicBoolean`, `ConcurrentHashMap`, `LinkedBlockingQueue`) — do NOT introduce coroutines outside the MCP package without discussion

## Code Style (ktlint)

**Plugin:** `org.jlleitschuh.gradle.ktlint` version `12.1.1`, ktlint version `1.5.0`
**Config:** `.editorconfig` (root)

Key `.editorconfig` settings:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [six2dez/burp-ai-agent](https://github.com/six2dez/burp-ai-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
