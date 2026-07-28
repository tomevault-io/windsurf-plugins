---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the New Relic Java Agent, a bytecode instrumentation agent that monitors Java applications at runtime. The agent attaches via the `-javaagent` JVM flag, uses ASM (version 9.9.1) for bytecode manipulation, and employs a "weaver" pattern to insert monitoring code into target classes as they are loaded by the JVM classloader.

The agent captures transactions, traces, metrics, errors, and logs from 440+ supported frameworks and libraries, sending telemetry data to the New Relic platform.

The product optimizes for:
- safety and stability above all else (should never crash the JVM being monitored)
- minimal impact on CPU and memory resources of the JVM being monitored
- backwards compatibility targeting the Java 8 runtime
- collection of application profiling data that is useful and accurate
- clarity over cleverness
- accessible object-oriented design

Avoid over-engineering. If a simpler solution exists, use it.

## Critical Constraints

These rules are non-negotiable:

- **Java 8 language level** for all production code (no newer language features)
- **Do NOT use lambda expressions in weaver instrumentation modules** — the weaver's bytecode rewriting cannot reliably transform lambdas due to `invokedynamic` bootstrap methods and classloader isolation; use anonymous classes instead
- **Do NOT change public APIs** without explicit instructions
- **Do NOT change weaver code** without explicit instructions
- **Preserve backward compatibility** for all shared components
- **Flag major architectural changes** before implementing — don't just do it
- **Instrumentation modules must be self-contained** — dependencies must be shaded
- **New dependencies require shading and licensing** — only add if they provide significant value and always ask for permission first

## Tech Stack

- Gradle for build and dependency management
- JUnit + Mockito for tests
- ASM 9.9.1 for bytecode manipulation
- Caffeine for high-performance caching
- Apache `HttpClient` for data transport to the New Relic backend
- Shadow Gradle plugin for relocating packages of project dependencies
- Spotbugs for static code analysis
- Jacoco for code coverage

## Key File Paths

| Entry Point           | Path                                                                              |
|-----------------------|-----------------------------------------------------------------------------------|
| Agent bootstrap       | `newrelic-agent/src/main/java/com/newrelic/bootstrap/BootstrapAgent.java`         |
| Configuration         | `newrelic-agent/src/main/java/com/newrelic/agent/config/AgentConfigImpl.java`     |
| Service registry      | `newrelic-agent/src/main/java/com/newrelic/agent/service/ServiceManagerImpl.java` |
| Weave engine          | `newrelic-weaver/src/main/java/com/newrelic/weave/ClassWeave.java`                |
| Weave manifest cache  | `gradle/script/cache_weave_attributes.gradle.kts`                                 |
| Code style definition | `dev-tools/code-style/java-agent-code-style.xml`                                  |

## Architecture

The agent uses Java's `Instrumentation` API (JSR 163) with two entry points in `BootstrapAgent`:

```
premain(String agentArgs, Instrumentation inst)   ← normal startup via -javaagent
agentmain(String agentArgs, Instrumentation inst)  ← dynamic attach
```

### Core Modules

| Module                   | Purpose                                                                                                                                                   |
|--------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------|
| `newrelic-agent`         | Main agent implementation; ServiceManager, configuration, harvest cycle, data transport. Built as a shadow JAR with all dependencies relocated.           |
| `newrelic-weaver`        | Bytecode weaving engine using ASM. Matches weave classes to target classes and applies transformations at class-load time.                                |
| `newrelic-weaver-api`    | Annotations for authoring weave classes (`@Weave`, `@NewField`, `@WeaveWithAnnotation`).                                      |
| `newrelic-api`           | Public API for custom instrumentation (`@Trace`, `NewRelic.getAgent()`, custom events/metrics). Ships with no-op implementations.                         |
| `agent-bridge`           | Runtime bridge between instrumentation modules and agent core. Provides `AgentBridge` static facade with volatile references swapped in when agent loads. |
| `agent-bridge-datastore` | Datastore-specific bridge interfaces (connection URL parsing, vendor detection, instance metrics).                                                        |
| `agent-interfaces`       | Internal agent interfaces shared across modules.                                                                                                          |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [newrelic/newrelic-java-agent](https://github.com/newrelic/newrelic-java-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
