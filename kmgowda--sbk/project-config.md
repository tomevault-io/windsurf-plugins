---
trigger: always_on
description: Copyright (c) KMG. All Rights Reserved.
---

<!--
Copyright (c) KMG. All Rights Reserved.
Licensed under the Apache License, Version 2.0.
-->

# AGENTS.md — AI Agent Guide for the SBK Repository

> **Universal entry point for all AI agents.** This file is the standard
> entry point for AI coding agents (Devin, Claude Code, Cursor, GitHub
> Copilot, Continue, Aider, OpenAI Codex, Windsurf, etc.) working in this
> repository. It tells the agent **what SBK is, how to build and verify it,
> what conventions to follow, where things live, and what the common
> gotchas are**.
>
> **Agent-specific configurations:**
> - Devin: See `.devin/skills/` for executable skills
> - Cursor: See `.cursorrules` for Cursor-specific rules
> - Aider: See `.aider.conf.yml` for Aider configuration
>
> **Humans:** see <ref_file file="/root/projects/SBK/README.md" /> for the
> end-user manual, and <ref_file file="/root/projects/SBK/docs/sbk-internals.md" />
> for the internal design.

---

## 1. What this repository is

**SBK** (Storage Benchmark Kit) is a Java framework for benchmarking *any*
S3-compatible / message-queue / file-system / database storage. It runs
identical measurement code against any backend via a small pluggable
driver SPI.

- **Languages.** Java only (no Kotlin, no Scala). Build with Gradle (wrapper
  in tree: `./gradlew`). JDK 25 required.
- **Modules.** 6 core modules + ~55 storage drivers. Each module is a
  Gradle subproject.
- **License.** Apache 2.0.
- **Branch model.** Trunk-based; PRs target `master`.

### Module map (memorise this)

| Module | Role | When you edit it |
|---|---|---|
| `perl/` | **PerL** — Performance Logger library (lock-free queues, latency windows, percentile math). Heart of the framework. | Rarely. Only when changing core measurement behaviour. |
| `sbk-api/` | The benchmark harness. Defines `Storage<T>` SPI, `RWLogger` SPI, `SbkBenchmark`, `Sbk` main bootstrap. | When changing the harness, CLI flags, the SPI, or how loggers work. |
| `sbm/` | **SBM** — Storage Benchmark Monitor (gRPC aggregator on port 9717). | When changing distributed aggregation. |
| `sbk-yal/` | YML-driven launcher (single-node). | Rarely. |
| `sbk-gem/` | **SBK-GEM** — SSH-based distributed launcher. | When changing the multi-host orchestration. |
| `sbk-gem-yal/` | YML-driven SBK-GEM. | Rarely. |
| `drivers/<name>/` | One subdirectory per storage backend. **~55 driver subprojects**, each a small wrapper around a vendor SDK. | When adding or fixing a driver. **This is the most common change.** |

**For new drivers, see <ref_file file="/root/projects/SBK/docs/DRIVER_SPECIFICATION.md" />
(spec template + worked example) and
<ref_file file="/root/projects/SBK/docs/AGENT_RECIPES.md" /> ("Add a storage
driver" recipe).**

---

## 2. Build, run, and verify

### Build commands

```bash
# Default build — runs compile + checkstyle + tests on every module.
./gradlew check

# Build the launchable scripts at ./build/install/sbk/bin/sbk
./gradlew installDist

# Build a single driver (much faster while iterating)
./gradlew :drivers:minio:check
./gradlew :drivers:minio:compileJava
```

### Run a benchmark from source

```bash
# Smoke-test against the public MinIO sandbox (no credentials needed)
./build/install/sbk/bin/sbk -class minio -writers 1 -size 100 -seconds 30

# Local file-system write benchmark (1 MiB records, 8 writers, 60 s)
./build/install/sbk/bin/sbk -class file -file /tmp/sbk.bin \
   -writers 8 -size 1048576 -seconds 60

# Help / list available drivers
./build/install/sbk/bin/sbk -help
```

### Verification — what counts as "done"

A change is **done** only after all of these succeed:

1. `./gradlew :drivers:<your-driver>:check` — module-level
2. `./gradlew check` — full project
3. `./gradlew installDist` — produces a working `sbk` script
4. The end-to-end smoke test above runs without errors

> **Driver changes:** also run the driver's CLI against a real backend
> (or `play.min.io` for S3) before declaring success. Compile-clean is
> necessary but not sufficient.

---

## 3. Repository conventions

### File-system conventions

| Path | Convention |
|---|---|
| `drivers/<name>/build.gradle` | Subproject build script. Declares the driver's vendor-SDK dependency. |
| `drivers/<name>/src/main/java/io/sbk/driver/<Name>/<Name>.java` | The `Storage<T>` impl. Class name = PascalCase of the driver dir name. **Match the package case** (e.g. `MinIO` driver dir → `io.sbk.driver.MinIO` package → `MinIO.java`). |
| `drivers/<name>/src/main/java/io/sbk/driver/<Name>/<Name>Writer.java` | The `Writer<T>` impl. |
| `drivers/<name>/src/main/java/io/sbk/driver/<Name>/<Name>Reader.java` | The `Reader<T>` impl. |
| `drivers/<name>/src/main/java/io/sbk/driver/<Name>/<Name>Config.java` | POJO holding driver-specific config. Bound from the properties file by Jackson. |
| `drivers/<name>/src/main/resources/<name>.properties` | Default values for every config field. |

**Driver discovery is by simple class name, case-insensitive**:
`-class minio` resolves to `io.sbk.driver.MinIO.MinIO`. The class name
must match the file/directory name (modulo case).

### Code conventions

- **Lombok** is available (`@Synchronized`, `@SuppressFBWarnings`, etc.).
  See <ref_file file="/root/projects/SBK/lombok.config" />.
- **Checkstyle is strict.** The most common violations a new driver
  triggers:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kmgowda/SBK](https://github.com/kmgowda/SBK) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
