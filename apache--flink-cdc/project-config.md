---
trigger: always_on
description: Licensed to the Apache Software Foundation (ASF) under one
---

<!--
Licensed to the Apache Software Foundation (ASF) under one
or more contributor license agreements.  See the NOTICE file
distributed with this work for additional information
regarding copyright ownership.  The ASF licenses this file
to you under the Apache License, Version 2.0 (the
"License"); you may not use this file except in compliance
with the License.  You may obtain a copy of the License at

  http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing,
software distributed under the License is distributed on an
"AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
KIND, either express or implied.  See the License for the
specific language governing permissions and limitations
under the License.
-->

# Flink CDC AI Agent Instructions

This file provides guidance for AI coding agents working with the Apache Flink CDC.

## Prerequisites

- Java 11 (baseline; all code must compile and run on Java 11) and Java 17
- Maven 3.8.6 or higher
- Git
- Unix-like environment (Linux, macOS, WSL)
- Docker (required for running integration tests and e2e tests)

## Commands

### Build

- Fast dev build (skip tests and format checks): `mvn clean install -DskipTests -Dspotless.check.skip=true -Dcheckstyle.skip=true`
- Full build against Flink 1.x (default): `mvn clean package -DskipTests`
- Full build against Flink 2.x: `mvn clean package -DskipTests -Pflink2`
- Single module (e.g. `flink-cdc-common`): `mvn clean package -DskipTests -pl flink-cdc-common -am`

### Testing

- Run all tests for a module: `mvn verify -pl flink-cdc-common`
- Run all tests against Flink 2.x: `mvn verify -pl flink-cdc-common -Pflink2`
- Run a single test class: `mvn -pl flink-cdc-common -Dtest=MyTest test`
- Run a single test method: `mvn -pl flink-cdc-common -Dtest=MyTest#myMethod test`

### Code Quality

- Before committing changes, run `mvn spotless:apply` and `mvn spotless:check` to enforce code style rules.
- Make sure newly added files have proper ASF license headers.

## Repository Structure

Flink CDC is organized around the Pipeline abstraction: a user-defined data pipeline that reads from one or more sources, optionally transforms records, and writes to one or more sinks. The core modules implement this abstraction; the connector modules provide the concrete source and sink implementations.

### Core Modules

- `flink-cdc-common` — Shared API and data model used across all modules: CDC event types (`DataChangeEvent`, `SchemaChangeEvent`, etc.), the schema model, data types, source/sink interfaces, `Factory` SPI, route definitions, UDF interfaces, and utility classes. Most new abstractions start here.
- `flink-cdc-runtime` — Runtime implementation of the Pipeline: operators for reading, routing, transforming (expression evaluation via Calcite + Janino), and writing CDC events.
- `flink-cdc-composer` — Pipeline assembly and deployment layer. Translates a `PipelineDefinition` into a runnable Flink job, wiring sources, operators, and sinks together. Supports Flink-native, Kubernetes, and YARN deployment.
- `flink-cdc-cli` — Command-line entry point (`flink-cdc.sh`). Parses YAML pipeline definitions and delegates to `flink-cdc-composer`.
- `flink-cdc-dist` — Distribution packaging. Produces the `flink-cdc-<version>-bin` release archive.

### Flink Version Compatibility

Currently, Flink CDC supports two Flink generations simultaneously:

- `flink-cdc-flink1-compat` — Flink 1.x compatibility layer (currently 1.20.3). Default profile.
- `flink-cdc-flink2-compat` — Flink 2.x compatibility layer (currently 2.2.0). Activated via `-Pflink2`.

All modules that depend on Flink APIs must declare their Flink dependencies as `provided` and reference `${flink.version}`, which is resolved by the active profile.
Please verify your changes in both Flink 1.20 (LTS) and Flink 2.x (latest).

### Connectors (`flink-cdc-connect/`)

Connectors are split into two categories:

- Source Connectors (`flink-cdc-connect/flink-cdc-source-connectors/`) are CDC sources for DataStream and Flink SQL jobs.
- Pipeline Connectors (`flink-cdc-connect/flink-cdc-pipeline-connectors/`) are Pipeline connectors for the YAML API.

### Tests

Write unit tests and integration tests in the corresponding submodules. End-to-end tests are located in these modules:

- `flink-cdc-e2e-tests/` — End-to-end tests parent module.
  - `flink-cdc-e2e-utils` — Shared test utilities (container management, assertions)
  - `flink-cdc-source-e2e-tests` — E2E tests for source connectors
  - `flink-cdc-pipeline-e2e-tests` — E2E tests for pipeline connectors

### Documentation

- `docs/` — Hugo-based documentation site. `docs/content/` for English, `docs/content.zh/` for Chinese. Update both when adding new features.

## Coding Standards

- **Format Java files with Spotless before every commit:** `mvn spotless:apply`.
- **Import order** (enforced by Checkstyle): `org.apache.flink.cdc` → `org.apache.flink` → other third-party → `javax` → `java`. Static imports go last. No star imports.
- **Forbidden imports** (enforced by Checkstyle):
  - JUnit 4 (`org.junit.*` except `org.junit.jupiter.*`) — use JUnit 5 Jupiter instead
  - `org.junit.jupiter.api.Assertions` and `org.hamcrest` — use AssertJ instead

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [apache/flink-cdc](https://github.com/apache/flink-cdc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
