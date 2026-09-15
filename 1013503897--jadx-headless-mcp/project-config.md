---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

A headless MCP (stdio) server for Android APK static analysis, written in Kotlin on top of the `jadx-core` library. Distinguishing trait vs. existing JADX MCP projects: **no JADX GUI / no plugin / no Python adapter** — one JVM process speaking MCP over stdio. The server exposes ~20 tools (`load_apk`, `get_class_source`, `get_xrefs_to_method`, etc.) and is registered once in MCP client config; APKs are swapped at runtime via the `load_apk` / `unload_apk` tools rather than by editing config.

Requires JDK 17+. Single Gradle module, no submodules.

Local checkout: `C:/work/git_code/hook-lab/jadx-headless-mcp`.  
MCP name `jadx-headless` → launcher `build/install/jadx-headless-mcp/bin/jadx-headless-mcp.bat` (Claude `~/.claude.json` / Grok `~/.grok/config.toml`).

## Build & run

```bash
./gradlew installDist
# launcher: build/install/jadx-headless-mcp/bin/jadx-headless-mcp[.bat]

./gradlew shadowJar
# fat jar: build/libs/jadx-headless-mcp-<version>-all.jar

./gradlew run          # runs main with stdin wired through (standardInput = System.in)
./gradlew compileKotlin # quickest type-check

# After installDist, smoke-test the launcher:
./build/install/jadx-headless-mcp/bin/jadx-headless-mcp --help
```

No test sources exist yet (`src/main/kotlin` only). CI (`.github/workflows/build.yml`) runs `installDist` + `--help` smoke check on ubuntu-latest.

## Architecture

```
stdin/stdout (MCP JSON-RPC)   ── OR ──   Ktor CIO HTTP (Streamable HTTP, --transport http)
        │
   Main.kt ── registers ~26 tools on a Server, then StdioServerTransport | mcpStreamableHttp
        │
   SessionHolder ── Mutex-serialized load() / unload(); holds 0..1 JadxSession
        │
   JadxSession ── thin wrapper over jadx.api.JadxDecompiler
                   • lazy `classes`, `resources`, FQN→class map, name→methods map
                   • truncate(maxSourceBytes) applied in getClassSource/getSmali
                   • decompilation is lazy: only get_class_source / get_smali_of_class
                     trigger per-class work; startup builds metadata indexes only
```

Three Kotlin files total; keep changes scoped to whichever layer owns the concern:

- `Main.kt` — arg parsing, MCP tool registration & input-schema definitions, JSON result helpers, AndroidManifest/strings parsing (regex-based, intentionally not a full XML parser), resource rendering. Adding a new MCP tool = adding a `server.addTool(...)` block here.
- `SessionHolder.kt` — thread-safe single-slot lifecycle (`load`, `unload`, `snapshot`, `current`). Always go through this rather than holding a `JadxSession` reference elsewhere.
- `JadxSession.kt` — all direct contact with `jadx.api.*`. New static-analysis primitives (finding nodes, describing xrefs, etc.) belong here so `Main.kt` stays a thin tool-dispatcher.

### Class resolution, inner classes & smali fallback (v0.3.0)

- **`decompiler.classes` is TOP-LEVEL ONLY.** Inner / `$Companion` / synthetic nested classes are reachable *only* via `JavaClass.getInnerClasses()`. `JadxSession.allClasses` flattens them recursively; the FQN/raw/canonical indexes and `methodsByName` are built from `allClasses`, which is what makes `Outer$Companion` addressable. If you add a new "find a class" path, resolve through `resolveClass` / `resolveClassDetailed`, never re-scan `classes` directly.
- **`resolveClass(query)`** accepts dotted (`Outer.Inner`), raw (`Outer$Inner`, `…$Companion`), canonical (`$`/`.` unified), and unique fuzzy suffix / simple-name forms. `resolveClassDetailed` returns candidates when ambiguous. All class-taking tools go through `Main.resolveClassArg` (rich not-found error with candidates).
- **Seamless smali fallback:** `getClassSourceSmart` / `getMethodBodySmart` detect jadx decompile-failure banners (`detectDecompileFailure`: strong markers like *"Code decompiled incorrectly"*, *"Method not decompiled"*, *"Can't load method instructions"*; soft: *"unreachable blocks"*). On a STRONG marker they return smali with a `// [jadx java-decompile failed → smali]` header (opt out via `smali_fallback=false`). jadx has **no per-method smali API** — `getMethodSmali` slices `.method`…`.end method` blocks out of the class disassembly (`JavaClass.getSmali()` / `ClassNode.getDisassembledCode()`).
- **New tools:** `get_method_body`, `get_method_smali`, `get_inner_classes`, `resolve_class`; enhanced: `get_class_source` + `get_method_by_name` (`smali_fallback`), `get_smali_of_class` (`offset` paging with `total_bytes`/`next_offset`). 26 tools total.

> **Resource export moved out (v0.5.0):** `export_apk_resources` / `export_arsc_resources` were removed from here and live in the sibling **`dex-forge`** MCP, so this server stays read-only analysis and the whole APK-repack pipeline (dex + resources) sits in one place. See `../dex-forge`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [1013503897/jadx-headless-mcp](https://github.com/1013503897/jadx-headless-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-15 -->
