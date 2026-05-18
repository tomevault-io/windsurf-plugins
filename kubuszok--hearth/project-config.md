---
trigger: always_on
description: This document provides guidelines for AI agents (like Claude Code, GitHub Copilot, Cursor, etc.) working with the Hearth codebase.
---

# AI Agent Guidelines for Hearth

This document provides guidelines for AI agents (like Claude Code, GitHub Copilot, Cursor, etc.) working with the Hearth codebase.

## Project Overview

**Hearth** is the first Scala macros' standard library, designed to make macro development easier and more maintainable across Scala 2.13 and Scala 3.

- **Language**: Scala (pure Scala library)
- **Scala Versions**: 2.13.16, 3.3.7 (primary); 2.13.18, 3.8.3 (regression testing)
- **Platforms**: JVM, Scala.js, Scala Native
- **Build Tool**: SBT (but see restrictions below)
- **License**: Apache 2.0

### Repository Structure

```
hearth/
├── hearth/                          # Main library module
│   └── src/main/
│       ├── scala/                   # Shared code (all versions/platforms)
│       ├── scala-2/                 # Scala 2.13-specific implementations
│       ├── scala-3/                 # Scala 3-specific implementations
│       ├── scalajs/                 # Scala.js platform code
│       ├── scalajvm/                # JVM platform code
│       └── scalanative/             # Scala Native platform code
│
├── hearth-better-printers/          # Better alternatives to showCode/showRaw
├── hearth-cross-quotes/             # Cross-platform quoting utilities
├── hearth-micro-fp/                 # Lightweight FP utilities
├── hearth-munit/                    # MUnit-based testing utilities
│
├── hearth-tests/                    # Test suite for main library
├── hearth-sandwich-tests/           # Cross-compilation tests (Scala 2 ↔ 3)
├── hearth-sandwich-examples-213/    # Scala 2.13 test cases
├── hearth-sandwich-examples-3/      # Scala 3 test cases
│
├── docs/                            # MkDocs documentation
├── project/                         # SBT build configuration
├── scripts/                         # Utility scripts
│
├── build.sbt                        # Main build configuration
├── dev.properties                   # Local IDE settings (DO NOT COMMIT)
├── CONTRIBUTING.md                  # Contribution guidelines
└── README.md                        # Project introduction
```

### Module dependencies

**Published modules:**
1. `hearth-better-printers` - Base printer utilities
2. `hearth-cross-quotes` - Core cross-platform quoting (depends on `hearth-better-printers`)
3. `hearth-micro-fp` - FP utilities (no dependencies)
4. `hearth` - Main library (depends on `hearth-micro-fp` and `hearth-better-printers`)
5. `hearth-munit` - MUnit integration (optional)

**Test modules:**
- `hearth-tests` - Main test suite (depends on `hearth`)
- `hearth-sandwich-tests` - Cross-compilation validation (depends on `hearth-tests`, both `compile` and `test` config)

### Development Configuration

The `dev.properties` file controls IDE settings:

```properties
# Choose IDE Scala version: 2.13 or 3
ide.scala = 3

# Choose IDE platform: jvm, js, or native
ide.platform = jvm

# Enable cross-quotes logging: true, false, or file names
log.cross-quotes = false
```

**Important:**
- DO NOT commit changes to `dev.properties`
- Consider running: `git update-index --assume-unchanged dev.properties`
- After changing settings, reload build in IDE

## Global rules

 - ✅ **allowed operations**:

    ```bash
    git status
    git diff
    git log
    git branch
    git show
    sbt --client "..."
    ```

 - ❌ **forbidden operations**:

    ```bash
    git commit
    git commit -m "message"
    git commit --amend
    git push
    git push --force
    git rebase
    git merge
    git reset --hard
    sbt          # bare sbt without --client is forbidden
    ```

 - **prefer working with MCP server** when available, it allows for:

    - **Compilation**: Request compilation through MCP tools
    - **Testing**: Run tests through MCP tools
    - **Diagnostics**: Get compiler errors, warnings, and type information
    - **Code navigation**: Find definitions, references, implementations
    - **Type information**: Query types at specific positions

 - **use `sbt --client` when MCP is insufficient** — e.g. when the task requires both Scala 2 and Scala 3
   (MCP exposes only 1 version at a time), or when MCP is down. Always use `--client` flag as sbt startup
   is expensive and kills the feedback loop — **never run bare `sbt` without `--client`**
 - **redirect sbt output to a temporary file** when running long compilation/test cycles — this avoids
   re-running the same expensive command just to inspect a different part of the output:
   ```bash
   sbt --client "quick-clean ; quick-test" 2>&1 | tee /tmp/sbt-output.txt
   ```
   Then use `grep`, `tail`, `head`, etc. on `/tmp/sbt-output.txt` to inspect results. Only re-run
   sbt if code was actually modified.
 - **do not modify `dev.properties`** - it's primarily used by the developer to focus on working on one platform in their IDE without the issues related to shared sources
   (IDE not being able to identify whether source file should be treated as a part of Scala 2 project or Scala 3 project, JVM or JS or Native)

### Finding the MCP Server Address

The MCP server configuration is stored in one of these files:
- `.metals/mcp.json` (standard Metals configuration)
- `.cursor/mcp.json` (Cursor IDE configuration)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kubuszok/hearth](https://github.com/kubuszok/hearth) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
