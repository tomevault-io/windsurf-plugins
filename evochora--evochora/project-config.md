---
trigger: always_on
description: Evochora is an artificial life simulator for research into digital evolution. It features:
---

# AGENTS.md

## Project Overview

Evochora is an artificial life simulator for research into digital evolution. It features:
- Custom n-dimensional simulation environment with thermodynamic constraints
- Multi-pass EvoASM compiler converting assembly to VM machine code
- High-performance runtime with embodied virtual organisms
- Modular data pipeline separating hot execution from cold data processing
- Web-based visualization and analysis frontends

## Repository Layout

- `src/main/java/` – Application code (compiler, runtime, datapipeline, CLI, node)
- `src/main/proto/` – Protobuf definitions for data pipeline communication
- `src/main/resources/` – Configuration files, compiler messages, reference.conf
- `src/test/java/` – Unit and integration tests
- `src/test/resources/` – Test resources and configurations
- `src/testFixtures/` – JUnit extensions and test utilities
- `docs/` – Documentation (ASSEMBLY_SPEC.md, CLI_USAGE.md, proposals)
- `assembly/` – Assembly code examples and test files
- `config/` – User-facing configuration (evochora.conf, local.conf)
- `build.gradle.kts` – Gradle build configuration
- `gradlew`, `gradlew.bat`, `gradle/wrapper/` – Gradle wrapper

## Build & Run (Java/Gradle)

**Java 21 required.** Configure in IDE or use `JAVA_HOME`.

```bash
./gradlew build              # Full build with tests
./gradlew clean assemble     # Assemble without tests
./gradlew test               # All tests
./gradlew unit               # Fast unit tests only (@Tag("unit"))
./gradlew integration        # Integration tests only (@Tag("integration"))
./gradlew run --args="node run"  # Run simulation node
./gradlew run --args="--help"    # Show CLI help
./gradlew distZip distTar    # Create distribution archives
```

## Running the Application

**Start the simulation node:**
```bash
./gradlew run --args="node run"
```

**With custom configuration:**
```bash
./gradlew run --args="--config my-config.conf node run"
```

**Show available commands:**
```bash
./gradlew run --args="--help"
```

**Get help for specific command:**
```bash
./gradlew run --args="help compile"
./gradlew run --args="help node"
```

### HTTP API for Pipeline Control

When the node is running, it exposes a REST API for controlling and monitoring the data pipeline:

**Pipeline-wide control:**
- `GET /api/pipeline/status` - Get overall pipeline status
- `POST /api/pipeline/start` - Start all services
- `POST /api/pipeline/stop` - Stop all services
- `POST /api/pipeline/restart` - Restart all services
- `POST /api/pipeline/pause` - Pause all services
- `POST /api/pipeline/resume` - Resume all services

**Individual service control:**
- `GET /api/pipeline/service/{serviceName}/status` - Get service status
- `POST /api/pipeline/service/{serviceName}/start` - Start specific service
- `POST /api/pipeline/service/{serviceName}/stop` - Stop specific service
- `POST /api/pipeline/service/{serviceName}/restart` - Restart specific service
- `POST /api/pipeline/service/{serviceName}/pause` - Pause specific service
- `POST /api/pipeline/service/{serviceName}/resume` - Resume specific service

## Assembly Compile System
The compiler can be invoked in multiple equivalent ways. For details and examples, see the **Compile** section in `docs/CLI_USAGE.md`.

- Primary user-facing entry point: `bin/evochora compile --source-root <root> --file=<path> [--env=<dimensions>[:<toroidal>]]`
- Developer entry point via JAR (after `./gradlew jar`):
  `java -jar build/libs/evochora.jar compile --source-root <root> --file=<path> [--env=<dimensions>[:<toroidal>]]`

Source roots define base directories (or HTTP URLs) from which module paths are resolved. Use `--source-root path:PREFIX` for named roots and `PREFIX:path` in directives to target them.

The compiler produces a JSON `ProgramArtifact` with machine code layout, labels, registers, procedures, environment properties, and source/ token maps that can be used for debugging and analysis.

## Configuration

### File Layout

- `config/evochora.conf` — User-facing experiment template (overrides selected defaults)
- `src/main/resources/reference.conf` — All defaults with documentation (embedded in JAR, loaded automatically)
- `config/local.conf` — Local development overrides (not checked in)

### Config Resolution Cascade (in `ConfigLoader.resolve()`)

1. `--config` CLI option
2. `-Dconfig.file` system property
3. `config/evochora.conf` in CWD
4. `APP_HOME/config/evochora.conf` (detected from JAR location)
5. Classpath `reference.conf` only (fallback)

### Tuning Profiles

Services share pipeline tuning parameters via HOCON substitutions:
- `pipeline.tuning = ${profiles.detailed}` (or `sampled`, `sparse`)
- Services consume via `${pipeline.tuning.samplingInterval}`, `${pipeline.tuning.insertBatchSize}`, etc.
- Override in experiment config or `local.conf`: `pipeline.tuning = ${profiles.sampled}`

### Config Priority (within a single file)

System properties > Environment variables > User config file > reference.conf defaults

## Agent Guidelines

There is a central document for AI agent guidelines that defines architectural principles and review standards. All agents, automated workflows, and human developers should adhere to these rules.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [evochora/evochora](https://github.com/evochora/evochora) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
