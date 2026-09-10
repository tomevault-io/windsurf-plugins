---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

mjprof is a command-line "monadic" jstack/thread-dump analyzer. A command is a chain of small composable steps (monads) — data source → filters/mappers → terminal/output. Steps are separated by `.` or spaces; a step's arguments are wrapped in `/.../` and separated by `,` (slashes are used because `()[]{}` are shell-special). Escape a literal comma inside args as `,,`.

Example: `jstack -l <pid> | mjprof contains/state,RUNNABLE/.tree`

`mjprof-core` also exposes a typed, thread-free Java API (`com.performizeit.mjprof.core.pipeline.Pipeline`) for embedding — see `README.md`'s "Java API" section.

## Modules

This is a two-module Maven reactor (root `pom.xml` is `packaging=pom`, `<modules>mjprof-core, mjprof-cli</modules>`):

- **`mjprof-core`** (`com.performizeit.mjprof.core.*`) — thread-dump parsing, the data model, every plugin implementation, the plugin type interfaces, the threaded `Pipe`/`Generator` plumbing, and the typed `Pipeline`/`PipelineRunner` API. **Zero external main-scope dependencies** — keep it that way; anything that pulls in a library goes in `mjprof-cli` instead.
- **`mjprof-cli`** (`com.performizeit.mjprof.cli.*`) — the expression DSL (`monads/MJStep`, `monads/Macros`, `internal.macros.properties`), the plugin registry (`monads/StepsRepository`, `plugin/PluginRepoBuilder`, `plugin/PluginRepositoryScanner`), `MJProf.main`, and the `native` Maven profile. Depends on `mjprof-core` plus the `reflections` library.

## Build & test

Compiler source/target is Java 17. `.sdkmanrc` pins `java=25.1.3-graalce` (GraalVM CE, latest GA) — `sdk env install` then `sdk env` to use it. Only `mvn -Pnative package` actually needs GraalVM; the jar build and tests run on any JDK 17+.

- All tests, both modules, from the repo root: `mvn test` (JUnit 5 / Jupiter).
- Single test class/method needs `-pl` to target a module, and the two modules are asymmetric. `mjprof-core` has no reactor dependency, so `mvn -pl mjprof-core test -Dtest=StackTreeTest` works standalone. `mjprof-cli` depends on `mjprof-core` through the reactor (it isn't `mvn install`-ed into the local repo), so a bare `-pl mjprof-cli` fails dependency resolution — it needs `-am` to build `mjprof-core` first; that in turn makes surefire run in `mjprof-core` too, which errors out because the named class doesn't live there, so also pass `-Dsurefire.failIfNoSpecifiedTests=false`: `mvn -pl mjprof-cli -am test -Dtest=MJProfTest#someMethod -Dsurefire.failIfNoSpecifiedTests=false`.
- Build fat jar: `mvn clean package` → `mjprof-cli/target/mjprof-cli-1.1.0-jar-with-dependencies.jar` (main class `com.performizeit.mjprof.cli.MJProf`).
- Native image: `mvn -Pnative package -pl mjprof-cli -am` (GraalVM `native-maven-plugin`; `-am` also builds `mjprof-core` first since the profile lives only in `mjprof-cli`).
- Run without full package: `mvn -pl mjprof-cli exec:java -Dexec.mainClass=com.performizeit.mjprof.cli.MJProf -Dexec.args="..."` (or run the jar directly).

Coverage via JaCoCo runs during the `test` phase in each module.

## Plugin (monad) system — the core abstraction

Every user-facing step is a **plugin** class annotated with `@Plugin(name=..., params={...}, category=..., description=...)` (`mjprof-core`'s `core/api/Plugin.java`). `category` is a `PluginCategory` enum value; the value determines which interface the class must implement and where MJProf places it in the pipeline:

| Category | Interface (`core/plugin/types/`) | Role |
|---|---|---|
| `DATA_SOURCE` | `DataSource` | Generates thread dumps (jmx, path, stdin, visualvm) |
| `FILTER` | `Filter` | Keep/drop whole threads |
| `SINGLE_THREAD_MAPPER` | `SingleThreadMapper` | Transform one `ThreadInfo` (trim stack, drop props) |
| `DUMP_REDUCER` | `DumpReducer` | Operate on a whole dump (group, merge) |
| `THREAD_INFO_COMPARTAOR` | `ThreadInfoComparator` | Sorting |
| `TERMINAL` | `Terminal` | Consume the stream (count, tree, flat, list) |
| `OUTPUTER` | `Outputer` | Write results (stdout, snapshot, gui) |

Implementations live under `mjprof-core`'s `core/plugins/<category>/`.

### Two execution runtimes share the same plugins

Every plugin instance is also a `PipeHandler<ThreadDump, ThreadDump>` (or the `ThreadInfo`-level equivalent), and there are **two** ways that gets driven:

1. **CLI**: `mjprof-cli`'s `MJProf.constructPlumbing` wraps each step in a `core/plumbing/Pipe` — a `Thread` with a `BlockingQueue` — chained via `setOutgoingPipe`; data sources become `core/plumbing/Generator` threads. Genuinely multi-threaded, one thread per stage.
2. **API**: `mjprof-core`'s `pipeline/PipelineRunner` drives the same plugin instances synchronously on the calling thread, no threads or queues, via `pipeline/Pipeline`'s typed builder methods.

**Consequence:** a new or changed plugin must keep working under both. Don't assume `handleMsg`/`handleDone` are only ever called from a `Pipe`'s thread — `PipelineRunner` calls them directly and reuses the same instance across the whole run.

### Plugin discovery is two-phase — this matters


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AdoptOpenJDK/mjprof](https://github.com/AdoptOpenJDK/mjprof) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
