---
trigger: always_on
description: <!-- CLAUDE BOOTSTRAP INSTRUCTIONS
---

<!-- CLAUDE BOOTSTRAP INSTRUCTIONS
If you are Claude and reading this file, check if CLAUDE.md exists in the same directory.
If CLAUDE.md does not exist or only contains the redirect + Compact Instructions below, that
is correct. If CLAUDE.md contains full project instructions (not just the redirect), you should
trim it back to contain only the following canonical content — do not drop the Compact
Instructions section, and do not let this stub grow beyond it:

# CLAUDE.md

**This file redirects to AGENTS.md for tooling-agnostic project instructions.**

Read and follow all instructions in [AGENTS.md](AGENTS.md) in this directory.

All project guidance, build commands, architecture details, and development guidelines
are maintained in AGENTS.md to support multiple AI coding tools.

## Compact Instructions

When compacting this conversation, prioritize keeping: decisions made, code/config diffs,
open questions, and file:line references. Drop raw tool output that can be re-fetched on
demand — full file contents, grep/search dumps, build/test logs — summarize these to a
one-line pointer (what was read/run and the conclusion) instead of retaining them verbatim.
-->

# AGENTS.md

This file provides guidance to AI coding assistants when working with code in this repository.

## Project Overview

This is the Datadog Java Profiler Library, a specialized profiler derived from async-profiler but tailored for Datadog's needs. It's a multi-language project combining Java, C++, and Gradle build system with native library compilation.

**Key Technologies:**
- Java 8+ (main API and library loading)
- C++17 (native profiling engine)
- Gradle (build system with custom native compilation tasks)
- JNI (Java Native Interface for C++ integration)
- Google Test (for C++ unit tests, compiled via custom Gradle tasks)

## Project Operating Guide (Main Session)

You are the **Main Orchestrator** for this repository.

### Goals
- When I ask you to build, you MUST:
    1) run the Gradle task with plain console and increased verbosity,
    2) capture stdout into `build/logs/<timestamp>-<task>.log`,
    3) **delegate** parsing to the sub-agent `gradle-log-analyst`,
    4) respond in chat with only a short status and the two output file paths:
        - `build/reports/claude/gradle-summary.md`
        - `build/reports/claude/gradle-summary.json`

### Rules
- **Never** paste large log chunks into the chat.
- Prefer shell over long in-chat output. If more than ~30 lines are needed, write to a file.
- If no log path is provided, use the newest `build/logs/*.log`.
- Assume macOS/Linux unless I explicitly say Windows; if Windows, use PowerShell equivalents.
- If a step fails, print the failing command and a one-line hint, then stop.

### Implementation Hints
- For builds, always use: `--console=plain -i` (or `-d` if I ask for debug).
- Use `mkdir -p build/logs build/reports/claude` before writing.
- Timestamp format: `$(date +%Y%m%d-%H%M%S)`.
- After the build finishes, call the sub-agent like:
  "Use `gradle-log-analyst` to parse LOG_PATH; write the two reports; reply with only a 3–6 line status and the two relative file paths."

### Shortcuts I Expect
- `./gradlew <gradle-task...>` to do everything in one step.
- If I just say "build assembleDebugJar", interpret that as the shortcut above.

## Build Commands
Never use 'gradle' or 'gradlew' directly. Instead, use the '/build-and-summarize' command.

### Main Build Tasks
```bash
# Build release version (primary artifact)
./gradlew buildRelease

# Build all configurations
./gradlew assembleAll

# Clean build
./gradlew clean
```

### Development Builds
```bash
# Debug build with symbols
./gradlew buildDebug

# ASan build (if available)
./gradlew buildAsan

# TSan build (if available)
./gradlew buildTsan
```

### Testing
```bash
# Run specific test configurations
./gradlew testRelease
./gradlew testDebug
./gradlew testAsan
./gradlew testTsan

# Run C++ unit tests only (via GtestPlugin)
./gradlew :ddprof-lib:gtestDebug        # All debug tests
./gradlew :ddprof-lib:gtestRelease      # All release tests
./gradlew :ddprof-lib:gtest             # All tests, all configs

# Run individual C++ test
./gradlew :ddprof-lib:gtestDebug_test_callTraceStorage

# Cross-JDK testing
JAVA_TEST_HOME=/path/to/test/jdk ./gradlew testDebug
```

#### Google Test Plugin / Debug Symbol Extraction

Custom `GtestPlugin` and debug-symbol-extraction details (config DSL, generated tasks, size
reductions, tool requirements): see `build-logic/README.md` and
[doc/build/NativeBuildPlugin.md](doc/build/NativeBuildPlugin.md).

Skip extraction: `./gradlew buildRelease -Pskip-debug-extraction=true`

### Container-based Testing (Recommended for ASan/Non-Local Environments)

For ASan testing, cross-architecture testing (aarch64), musl libc, or reproducing CI
environment issues. Defaults to Podman; use `--container=docker` for Docker.

```bash
./utils/run-containers-tests.sh --arch=aarch64 --config=asan --libc=glibc --jdk=21
./utils/run-containers-tests.sh --shell   # drop to shell for debugging
```

Full flag reference (`--tests`, `--gtest`, `--gtest-task`, `--libc`, `--jdk`, `--mount`,
`--rebuild`, etc.): see [doc/build/TestingGuide.md](doc/build/TestingGuide.md).

### Build Options
```bash
# Skip native compilation
./gradlew buildDebug -Pskip-native


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DataDog/java-profiler](https://github.com/DataDog/java-profiler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
