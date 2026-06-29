---
trigger: always_on
description: This document provides guidance for AI assistants working on the BitcoinAddressFinder codebase.
---

# CLAUDE.md — BitcoinAddressFinder

This document provides guidance for AI assistants working on the BitcoinAddressFinder codebase.

---

## Project Overview

**BitcoinAddressFinder** is a high-performance, open-source tool for scanning Bitcoin and altcoin private keys and checking whether the derived addresses have balances. It supports 100+ cryptocurrencies and uses both CPU (Java) and GPU (OpenCL) key generation strategies.

- **Group ID:** `net.ladenthin`
- **Artifact ID:** `bitcoinaddressfinder`
- **Version:** 1.6.1
- **Java:** 21
- **License:** Apache 2.0
- **Author:** Bernard Ladenthin (Copyright 2017–2025)
- **Main class:** `net.ladenthin.bitcoinaddressfinder.cli.Main`

---

## Build System

The project uses **Maven** (minimum 3.6.3) with a Maven Wrapper.

### Common Commands

```bash
# Compile only
./mvnw compile

# Run all tests
./mvnw test

# Build fat JAR with all dependencies
./mvnw package -P assembly

# Build without tests
./mvnw package -DskipTests

# Generate coverage report
./mvnw test jacoco:report

# Skip LMDB-specific tests (useful when LMDB native libs unavailable)
./mvnw test -Dnet.ladenthin.bitcoinaddressfinder.disableLMDBTest=true
```

### JVM / Compiler Flags

Tests run with `-Xmx2g` (no eager `-Xms` — forks are short-lived, one fresh JVM per test class with `reuseForks=false`, so an eager initial heap only inflated the spawn-time footprint) and several `--add-opens` / `--add-exports` to allow LMDB and internal module access. These are configured in `pom.xml` (`<argLine>`) and `.mvn/jvm.config`.

**Error Prone** static analysis with **NullAway** is active at compile time:
- All code in `net.ladenthin` packages must carry proper `@Nullable` / `@NonNull` annotations.
- Compilation will fail on potential null-pointer issues unless annotated.

### Verifying Javadoc locally (release builds)

The javadoc jar is attached only by the publish/deploy job (`mvn -P release deploy`).
So a javadoc break is **invisible to `mvn test` and to PR CI** and only fails the
snapshot publish on `main`. To reproduce the deploy-time javadoc step locally, run the
**full lifecycle**:

```bash
mvn -P release clean package -DskipTests -Dgpg.skip=true \
    -Dnet.ladenthin.bitcoinaddressfinder.disableLMDBTest=true
# expected: BUILD SUCCESS + target/*-javadoc.jar
```

Do **not** rely on a standalone `mvn javadoc:jar` — it runs before
`target/classes/module-info.class` exists, so javadoc takes the classpath-mode path
and cannot reproduce the JPMS module-mode failure that only appears in the full
build. `attach-javadocs` is bound to `prepare-package` and the `maven-javadoc-plugin`
block is declared **before** `maven-compiler-plugin` on purpose, so javadoc runs while
`target/classes/` is still module-descriptor-free and stays in classpath mode (module
mode is unusable here — the module declares no `requires` and `module-info.java` lives
in `src/main/java9`, off javadoc's source path). See the extensive comments on those two
`pom.xml` executions before touching their phase or ordering.

**The single surviving `-Dmaven.javadoc.skip=true` in CI (BAF-only, do not remove blindly).**
After a cross-repo cleanup that deleted every other javadoc-skip flag from all four sibling
pipelines, exactly **one** remains: the `build` job in `.github/workflows/publish.yml`. It is
necessary because that job runs plain `mvn package`, which triggers `attach-javadocs` at
`prepare-package`; BAF's javadoc is module-aware (`<source>21</source>` + `module-info.java`
in `src/main/java9`), so an unguarded run can flip into JPMS module mode and fail with
`No source files for package net.ladenthin...`. The Java-8 siblings (java-llama.cpp,
streambuffer, llamacpp-ai-index — all `<source>8>`) stay in classpath mode regardless and
therefore carry **no** skip flag; the redundant/no-op flags they used to have were removed,
and streambuffer/llamacpp let javadoc build during their `verify` test job so it is gated in
PR CI. BAF cannot do the same cheaply (the module-mode trap needs the full `-P release
package` ordering), which is why BAF's javadoc is validated only at publish. Before dropping
this last flag, prove `mvn package` builds BAF's javadoc clean — a standalone `mvn
javadoc:jar` cannot, it always runs classpath mode and hides the trap.

### JPMS module descriptor (`module-info.java`) handling — why it's hidden until the end

BAF ships a JPMS module descriptor (module `net.ladenthin.bitcoinaddressfinder`, exports
every package, `requires static lombok`, opens `configuration` to Jackson) **but goes to
unusual lengths to keep `module-info` invisible to every build tool until the last moment.**
This is the single most surprising part of the build; the authoritative rationale lives in
the `pom.xml` comments on the `module-info-compile` and `default-testCompile` executions and
the `maven-javadoc-plugin` block — read those before touching any of it. Summary:

- **`module-info.java` lives in `src/main/java9`, not `src/main/java`.** If it were on the
  main source path, `javac` would auto-flip the *whole* main compile into module mode, which
  (a) breaks Error Prone / NullAway / Checker Framework and (b) forbids the system-module

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bernardladenthin/BitcoinAddressFinder](https://github.com/bernardladenthin/BitcoinAddressFinder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
