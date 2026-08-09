---
trigger: always_on
description: Instructions for AI coding agents working in the OSHI repository. Humans should start with
---

# AGENTS.md

Instructions for AI coding agents working in the OSHI repository. Humans should start with
[README.md](README.md) and [CONTRIBUTING.md](CONTRIBUTING.md); this file covers what an agent
is likely to get wrong.

## What OSHI is

A cross-platform Java library that reads operating system and hardware information through native
APIs, with no bundled native libraries. It supports Windows, macOS, Linux/Android, and six UNIX
variants (AIX, DragonFly BSD, FreeBSD, NetBSD, OpenBSD, Solaris/illumos), through two independent
native-access implementations (JNA and the JDK Foreign Function & Memory API).

**The single most important consequence: you can only build and test the platform you are running
on, and only against the hardware you happen to have.** Most changes in this repository cannot be
verified by running them. Plan for that instead of assuming a green local build means the change is
correct.

## Repository map

| Path | Contents |
|---|---|
| `oshi-common` | Public API interfaces, abstract bases, POJOs, shared parsing/util code, and a pure-Java (no native access) implementation under `oshi.nativefree` for Linux and NetBSD. **Java 8; no JNA, no FFM.** |
| `oshi-core` | The JNA implementation. **Java 8.** |
| `oshi-core-ffm` | The FFM implementation. **Java 25+.** |
| `oshi-demo` | Proof-of-concept examples, jbang catalog entries. |
| `oshi-metrics` | Micrometer bindings. Java 17+. |
| `oshi-benchmark` | JMH benchmarks and the `oshi.comparison` cross-implementation tests. Java 25+. |
| `oshi-dist` | Distribution zip assembly (a single `.zip`, excluding `oshi-demo`). |
| `config/` | Checkstyle, import-control, forbidden-apis, license header, formatter, Sonar config. |
| `src/site/` | Maven site sources (`SampleOutput.md`, project listings). |

Within `oshi-core` and `oshi-core-ffm` the layout mirrors each other:

- `oshi/hardware/platform/<os>/` and `oshi/software/os/<os>/` — the per-OS implementations of the
  public interfaces.
- `oshi/driver/<os>/` — reusable helpers that fetch and parse one specific thing (a sysfs file, a
  WMI query, a `sysctl` call). Most parsing logic belongs here, not in the HAL classes.
- `oshi/jna/platform/<os>/` (JNA) and `oshi/ffm/platform/<os>/` (FFM) — the native mappings.
- `oshi/util/` — shared utilities. `ParseUtil`, `ExecutingCommand`, `FileUtil`, `GlobalConfig`.

### Where code belongs

`oshi-common` is not just the interfaces — it holds the shared *implementation*. Only code that
actually touches JNA or FFM types belongs in `oshi-core` / `oshi-core-ffm`.

Anything else — parsing, unit conversion, string handling, sorting and filtering, caching, sentinel
and range checks, the logic that turns raw native output into a returned value — belongs in
`oshi-common`, as an abstract base class the platform implementations extend, a `driver`/`util`
class both call, or a POJO they populate. If you are writing the same logic in both twins, it is in
the wrong place.

Reducing this duplication has been a sustained effort in the project, so **default to hoisting**
rather than copying. A change that adds parallel logic to both implementations will be asked to
justify why it is not shared.

**Before writing any parsing helper, check whether `ParseUtil` already has one.** It is the home for
general parsing primitives — numbers, sizes, frequencies, hex, delimited fields, and dates
(`parseDateToEpoch`, `parseCimDateTimeToOffset`, `parseYearlessDateToEpoch`). The duplication is
easy to miss because the two copies live in per-platform drivers that you would never read side by
side: the AIX and generic-UNIX `who` readers each grew their own copy of "parse a `MMM d HH:mm`
timestamp that carries no year, defaulting to the current year and subtracting one if that lands in
the future" before it was hoisted. If a command on one OS emits a format, a command on another
probably does too, so put the format parser in `ParseUtil` and leave only the
command-specific regex in the driver.

Runtime configuration lives in `oshi-common/src/main/resources/oshi.properties`, read through
`oshi.util.GlobalConfig` or overridden with Java system properties. Configuration is read at
startup, is not thread-safe, and is not re-read during operation. Add new settings there rather
than introducing a separate mechanism.

## The feedback loop

```sh
./mvnw clean install                 # full reactor build + tests
./mvnw -pl oshi-core -am test        # one module and its dependencies
./mvnw spotless:apply                # formatting, import order, license-header years
./mvnw checkstyle:check              # includes import-control
./mvnw forbiddenapis:check           # see "Banned APIs" below
./mvnw javadoc:javadoc               # javadoc errors fail the release build
```

⚠️ **`forbiddenapis:check` and `javadoc:javadoc` need compiled classes, so run them after `install`
in the same command, never after a bare `clean`.** `./mvnw clean javadoc:javadoc` fails with
`Creating an aggregated report for both named and unnamed modules is not possible` — `clean` deletes
`oshi-common/target/classes/module-info.class`, no standalone goal regenerates it, and the
aggregator then sees one unnamed module among named ones. The error names `oshi-common` and looks

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [oshi/oshi](https://github.com/oshi/oshi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
