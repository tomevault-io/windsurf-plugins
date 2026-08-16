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

# Flink Kubernetes Operator AI Agent Instructions

This file provides guidance for AI coding agents working with the Apache Flink
Kubernetes Operator codebase.

## Prerequisites

- Java 17 (the build compiles and targets Java 17; see `maven.compiler.source`/`target` in the root `pom.xml`)
- Maven 3.8.6 or later. There is **no** Maven wrapper in this repository — use the system `mvn`.
- Git
- Docker (to build operator images and run Testcontainers-based integration tests)
- A local Kubernetes cluster (`minikube` or `kind`), `kubectl`, and `helm` to deploy the operator and run end-to-end tests
- Unix-like environment (Linux, macOS, WSL)
- This operator builds against Apache Flink `1.20.x` (`flink.version` in the root `pom.xml`)

## Commands

### Build

- Fast dev build: `mvn clean install -DskipTests -T1C`
- Full build with tests: `mvn clean verify`
- Single module: `mvn clean install -DskipTests -pl flink-kubernetes-operator -am`
- Single module with tests: `mvn clean verify -pl flink-kubernetes-operator -am`
- Build the operator Docker image: `docker build -t flink-kubernetes-operator .`

### Testing

- Single test class: `mvn -pl flink-kubernetes-operator -Dtest=FlinkConfigManagerTest test`
- Single test method: `mvn -pl flink-kubernetes-operator -Dtest=FlinkConfigManagerTest#testConfigUpdate test`
- End-to-end tests are shell scripts under `e2e-tests/` (e.g. `test_application_operations.sh`); they require a running Kubernetes cluster (minikube) with the operator installed via the Helm chart in `helm/`.

### Code Quality

- Format code: `mvn spotless:apply` (google-java-format, **AOSP** style; version pinned in the root `pom.xml`)
- Check formatting: `mvn spotless:check`
- Checkstyle: `mvn checkstyle:check` (config: `tools/maven/checkstyle.xml`, suppressions: `tools/maven/suppressions.xml`)
- Apache license header check (Rat) runs as part of the build.
- Spotless-check and checkstyle are bound to the `validate` phase, so any build (even `-DskipTests`) fails on unformatted or non-compliant code — run `mvn spotless:apply` before building.

### Documentation

- Regenerate the configuration reference docs: `mvn clean package -pl flink-kubernetes-docs -am -Pgenerate-docs`
  (runs `ConfigOptionsDocGenerator`; output lands under `docs/layouts/shortcodes/generated`). Do not hand-edit generated reference files.

## Repository Structure

Every module from the root `pom.xml`, plus the supporting directories.

### Modules

- `flink-kubernetes-operator-api` — Custom Resource (CRD) types: `FlinkDeployment`, `FlinkSessionJob`, `FlinkStateSnapshot`, `FlinkBlueGreenDeployment` and their specs/status. **This is the user-facing public API surface** (`CustomResourceDescriptors`).
- `flink-kubernetes-operator` — The operator itself: JOSDK controllers, observers, reconcilers, the Flink service abstraction, validation, mutation, metrics, and config. The primary module.
- `flink-kubernetes-standalone` — Standalone-mode (non-native) Kubernetes deployment support used by the operator.
- `flink-kubernetes-webhook` — Admission webhook for validating and mutating custom resources.
- `flink-autoscaler` — Generic Flink job autoscaler, decoupled from Kubernetes so it can be reused outside the operator.
- `flink-autoscaler-standalone` — Runs the autoscaler as a standalone process against existing Flink clusters.
- `flink-autoscaler-plugin-jdbc` — JDBC-backed state store / event handler plugin for the autoscaler.
- `flink-kubernetes-docs` — Documentation build module; auto-generates the config and CRD reference.
- `examples/` — Example projects (`flink-sql-runner-example`, `flink-beam-example`, `kubernetes-client-examples`, `autoscaling`).

### Supporting directories

- `helm/` — Helm chart used to install the operator.
- `e2e-tests/` — End-to-end test scripts run against a real cluster.
- `docs/` — User-facing documentation (Hugo site) and generated shortcodes.
- `tools/` — Maven config (`checkstyle.xml`, `suppressions.xml`), license tooling, OLM, and release scripts.
- `.github/` — CI workflows and the PR template.

### Key packages in `flink-kubernetes-operator`

- `controller/` — JOSDK `Reconciler` entry points, one per CRD (`FlinkDeploymentController`, `FlinkSessionJobController`, `FlinkStateSnapshotController`, `FlinkBlueGreenDeploymentController`).
- `observer/` — Read the actual state of Flink jobs and Kubernetes resources (`deployment/`, `sessionjob/`, `snapshot/`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [apache/flink-kubernetes-operator](https://github.com/apache/flink-kubernetes-operator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
