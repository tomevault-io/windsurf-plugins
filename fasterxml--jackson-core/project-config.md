---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository
on branches of the 3.x line (`3.1`, `3.2`, `3.x`).

## Project

`jackson-core` is the streaming (incremental) parser/generator layer of Jackson, plus the reference
JSON implementation of those abstractions. Maven coordinates are `tools.jackson.core:jackson-core`
and all packages live under `tools.jackson.core` (2.x used `com.fasterxml.jackson.core`) — never
introduce `com.fasterxml.*` into 3.x code.

Its one compile dependency, `fastdoubleparser`, is shaded and relocated into
`tools.jackson.core.internal.shaded.fdp` at package time and stripped from the published POM — so
**consumers see zero external dependencies**. (Unlike 2.x, the relocation path is *not*
version-suffixed: JPMS module encapsulation already prevents reuse by downstream deps.)

Everything else in Jackson (`jackson-databind`, and every data format backend: Smile, CBOR, XML,
CSV, YAML, Protobuf...) builds on the abstractions here. That means **public API changes here ripple
across the whole ecosystem** — treat `JsonParser`, `JsonGenerator`, `TokenStreamFactory`,
`JsonFactory`, and the `*Feature` enums as frozen in patch releases and additive-only in minor ones.

## Build & test

Use the Maven wrapper (`./mvnw`), not a system `mvn`.

```bash
./mvnw verify                       # full build: compile, test, JaCoCo agent + enforcer checks
./mvnw test                         # tests only
./mvnw -B -ff -ntp verify           # exactly what CI runs (batch, fail-fast, no transfer log)

./mvnw test -Dtest=UTF8StreamJsonParserTest              # single test class
./mvnw test -Dtest=UTF8StreamJsonParserTest#testFoo      # single test method
./mvnw test -Dtest='*Filtering*'                         # pattern

./mvnw test jacoco:report                   # coverage report (separate goal; not part of `verify`)
./mvnw clean package animal-sniffer:check   # verify Android SDK 26 API compatibility
```

Note `verify` binds the JaCoCo *agent* and an enforcer rule that requires `jacoco.exec` to exist —
it does not produce the coverage report. CI generates that in a separate `test jacoco:report` step,
and runs `animal-sniffer:check` only on the release-build matrix entry.

Baseline is **JDK 17** (source/target); CI builds on 17, 21, and 25 (plus 17 on Windows). Normal
development works on JDK 17, but *releases* must be built with **JDK 21+** — the enforcer plugin
fails otherwise, because JDK 17 produces an incomplete `-javadoc.jar` (see #1561 / #1625).
`animal-sniffer` additionally restricts you to the Android SDK 26 API subset.

Tests are JUnit 5 (`org.junit.jupiter`) with AssertJ available. `src/test/java/perf/` holds manual
benchmark drivers, not unit tests.

## Branching and release notes

This repo maintains many live branches. Fixes go to the **oldest branch that should receive them**,
then get merged forward:

```
2.21 → 2.22 → 2.x → 3.1 → 3.2 → 3.x
```

`2.x` merges into `3.1`, so 2.x fixes flow into the 3.x line. `3.1` is the current patch branch
(3.1.6-SNAPSHOT), `3.2` the current minor branch (3.2.1-SNAPSHOT), `3.x` the dev branch
(3.3.0-SNAPSHOT). The `3.0` branch still exists (3.0.5-SNAPSHOT) but is dormant — fully merged into
`3.1`, no commits since 3.0.4 shipped, and no unreleased section in `release-notes/VERSION`. Start
from `3.1`, not `3.0`, unless told otherwise.

Don't commit a fix only to `3.x` if it belongs in a patch branch. Propagate with forward merges
(`git merge 3.1` into `3.2`, then `3.2` into `3.x`) rather than cherry-picks — that is the pattern
throughout the history.

Every user-visible change gets an entry in `release-notes/VERSION` under the unreleased version,
formatted as `#<issue>: <summary>` with `(reported by @user)` / `(contributed by @user)` lines, and
a matching name in `release-notes/CREDITS`. (`VERSION-2.x` / `CREDITS-2.x` are the frozen 2.x
history; do not add to them.)

## What changed from 2.x (things to unlearn)

- **`JacksonException` extends `RuntimeException`.** Nothing in the streaming API declares
  `throws IOException`; parser/generator methods declare `throws JacksonException`. Low-level I/O
  failures are wrapped as `exc/JacksonIOException`.
- **Factories are immutable.** There is no `factory.enable(...)` / `factory.configure(...)`.
  Configure through `JsonFactory.builder()` (or `factory.rebuild()` to derive a modified copy).
  `JsonFactory.builderWithJackson2Defaults()` moves defaults back toward 2.x, but its own Javadoc
  says it is a work in progress and does not yet fully replicate them — don't treat it as exact.
- **`JsonParser.Feature` and `JsonGenerator.Feature` are gone.** Their members were split into
  `StreamReadFeature`/`StreamWriteFeature` (format-neutral) and `JsonReadFeature`/`JsonWriteFeature`
  (JSON-only). `JsonFactory.Feature` moved up to `TokenStreamFactory.Feature`.
- **Renames**: `JsonLocation` → `TokenStreamLocation`, `JsonStreamContext` → `TokenStreamContext`,
  `JsonGeneratorImpl` → `json/JsonGeneratorBase`, `NonBlockingJsonParser` →
  `json/async/NonBlockingByteArrayJsonParser`. `nextFieldName()` and friends are `nextName()`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FasterXML/jackson-core](https://github.com/FasterXML/jackson-core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
