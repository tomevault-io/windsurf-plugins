---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

LeakCanary MCP Server is a Kotlin/JVM MCP (Model Context Protocol) server that automates Android memory leak detection. It reads LeakCanary output from `adb logcat` (primary) or the app's stored LeakCanary database (fallback), parses leak traces, classifies root causes, assigns priority, suggests fixes, and tracks leak history. Communication with AI clients (Cursor, VS Code, Android Studio) is via STDIO JSON-RPC.

## Build & Run Commands

```bash
# Build the fat JAR (includes all dependencies)
./gradlew shadowJar
# Output: build/libs/leakcanary-mcp-server-all.jar

# Run the server directly
java -jar build/libs/leakcanary-mcp-server-all.jar

# Build only (compile check)
./gradlew build

# Clean build
./gradlew clean shadowJar
```

There are no tests in this project currently.

## Architecture

**Entry point:** `Main.kt` creates a `Server`, registers 12 tools, and starts a `StdioServerTransport`.

**Source root:** `src/main/kotlin/com/leakcanary/mcp/`

**Data flow:** ADB (logcat or app storage) -> Parser -> Analyzer -> Tool response (JSON)

### Key Layers

- **`adb/AdbExecutor`** — Singleton that shells out to `adb` via `ProcessBuilder`. Auto-resolves adb path from `ANDROID_HOME`, `ANDROID_SDK_ROOT`, common SDK locations, or PATH. Provides `captureLeakLogs()`, `captureFullLogs()`, `listDevices()`, `clearLogcat()`, and `runShellCommand()`.

- **`adb/AppStorageReader`** — Reads LeakCanary's persisted leak data directly from the app's internal SQLite database (`databases/leaks.db` or `databases/leakcanary`). Pulls the DB to the host via `adb shell run-as <package> cat databases/<db>`. Primary method: deserializes the `HeapAnalysisSuccess` Java objects stored as BLOBs in the `heap_analysis.object` column using shark library — this provides full reference chains, retained sizes, leak status per node, GC roots, and heap metadata. Falls back to SQL-based queries if BLOB deserialization fails. Also provides `discoverLeakCanaryApps()` to scan all third-party packages for LeakCanary databases. Returns `StoredLeakResult` with traces and optional `HeapSummary`. Requires debuggable app.

- **`parser/LeakTraceParser`** — State-machine parser for LeakCanary's ASCII box-drawing format (unicode chars: `┬───`, `├─`, `│ ↓`, `╰→`). Extracts `LeakTrace` objects with signature, GC root, reference chain nodes, retained size, and leak status.

- **`parser/HeapSummaryParser`** — Parses the METADATA section from logcat for heap stats (class count, instance count, bitmap info, device info).

- **`analyzer/LeakAnalyzer`** — Classifies leaks by pattern (SINGLETON_LEAK, LISTENER_LEAK, CONTEXT_LEAK, LIBRARY_LEAK, VIEWMODEL_LEAK, HANDLER_LEAK) using heuristics on the reference chain text. Assigns priority: P0 (>20MB), P1 (>5MB), P2 (rest).

- **`analyzer/FixSuggester`** — Generates classification-specific fix suggestions (e.g., singleton -> use applicationContext, listener -> unregister in onDestroy).

- **`history/LeakHistoryStore`** — Persists leak records to `~/.leakcanary-mcp/history.json`. Tracks first/last seen timestamps and occurrence count per signature.

- **`tools/*`** — Each file registers one MCP tool on the `Server` via extension function `Server.register*Tool()`. 12 tools total: `detect_leaks`, `list_leaks`, `search_leak`, `get_heap_summary`, `get_leak_history`, `analyze_leak`, `suggest_fixes`, `clear_leaks`, `list_devices`, `leak_diff`, `get_device_memory`, `export_report`.

- **`tools/LeakSource.kt`** — Three-tier fetch strategy used by most tools. Tries logcat first; if empty and `package_name` provided, reads from app storage; if empty and no `package_name`, auto-discovers LeakCanary apps on the device. Returns a `LeakFetchResult` with the data source indicator and any discovered apps.

- **`tools/PackageFilter.kt`** — Multi-app support utilities. Extracts app package names from leak traces, detects when multiple apps have leaks, filters traces by package, and builds a user-facing prompt listing detected apps. `filterByPackageIfNeeded()` skips filtering when data comes from app storage (since class names are simple, not fully qualified).

- **`tools/DeduplicateLeaks.kt`** — Groups duplicate leak traces by signature, keeps the richest trace as representative, and tracks occurrence count within a single scan.

- **`model/*`** — `@Serializable` data classes: `LeakTrace`, `LeakNode`, `LeakStatus`, `HeapSummary`, `LeakReport`, `LeakSummary`, `LeakHistoryEntry`.

### Adding a New MCP Tool

1. Create `tools/NewTool.kt` with a `Server.registerNewTool()` extension function
2. Use `addTool(name, description, inputSchema) { request -> ... }` to define the tool
3. Register it in `Main.kt` by calling `server.registerNewTool()`
4. Use `fetchLeakTraces()` from `LeakSource.kt` for the two-tier data strategy
5. Use `deduplicateTraces()` if the tool lists multiple leaks

### Database-First Data Strategy

Tools use `fetchLeakTraces()` from `LeakSource.kt` which:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ravisharma46/Leak-Canary-Mcp-Server](https://github.com/ravisharma46/Leak-Canary-Mcp-Server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
