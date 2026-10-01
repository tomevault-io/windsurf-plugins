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

# Flink Kafka Connector AI Agent Instructions

This file provides guidance for AI coding agents working with the Apache Flink Kafka connector codebase.

## Prerequisites

- Java 11, 17, or 21. Java 11 syntax must be used everywhere: the build compiles with source level 11 (target 17). On JDK 11 pass `-Pjava11-target`, as CI does.
- Maven 3.8.6 (Maven wrapper `./mvnw` included; prefer it)
- Git
- Docker (every `*ITCase` starts Kafka through Testcontainers)
- Python 3.9 to 3.11 with tox, only for `flink-python`
- Unix-like environment (Linux, macOS, WSL)
- The connector builds against `flink.version` in the root `pom.xml`, which is the lowest supported Flink minor version. A change must also work against every Flink version in `.github/workflows/push_pr.yml` and the `main` rows of `.github/workflows/weekly.yml`.

## Commands

### Build

- Build without tests: `./mvnw clean install -DskipTests`
- Full build with tests: `./mvnw clean verify`
- Build against another Flink version (what CI does): `./mvnw clean install -DskipTests -Dflink.version=<version>`
- Single module: `./mvnw clean install -DskipTests -pl flink-connector-kafka`
- `-Dfast` skips RAT, checkstyle, spotless, enforcer and javadoc, but not japicmp in this repository.
- Dependency convergence (CI runs this on PRs): `./mvnw clean install -DskipTests -Pcheck-convergence -Dflink.convergence.phase=install`

### Testing

- `*Test` classes run in the `test` phase, everything else (`*ITCase`) in the `integration-test` phase through a second Surefire execution; there is no Failsafe plugin.
- Single unit test class: `./mvnw test -pl flink-connector-kafka -Dtest=KafkaSourceReaderTest`
- Single test method: `./mvnw test -pl flink-connector-kafka -Dtest=KafkaSourceReaderTest#testCommitOffsetsWithoutAliveFetchers`
- Single ITCase: `./mvnw test -pl flink-connector-kafka -Dtest=KafkaSinkITCase` (`-Dtest` overrides the phase filter; `verify -Dtest=...` runs the class twice)
- ArchUnit rules: `./mvnw test -pl flink-connector-kafka -Dtest='*ArchitectureTest'`, then check `git status flink-connector-kafka/archunit-violations/` (see Testing Standards)
- End-to-end tests need a Flink distribution: `./mvnw clean verify -Prun-end-to-end-tests -DdistDir=<path to flink-<version>>`. CI runs them on every PR.
- PyFlink tests: `./mvnw clean install -DskipTests`, then `cd flink-python && chmod a+x dev/* && ./dev/lint-python.sh -e mypy,sphinx`. The scripts are downloaded into `flink-python/dev/` during the Maven `validate` phase without the executable bit, which is why CI sets it first.
- CI is defined by `apache/flink-connector-shared-utils` (`.github/workflows/ci.yml@ci_utils`); its Maven command line, including the license check, is the reference when a local run differs from CI.

### Code Quality

- Format code: `./mvnw spotless:apply` (skipped automatically on JDK 21; run it on 11 or 17)
- Check formatting: `./mvnw spotless:check`
- Checkstyle: `./mvnw checkstyle:check`
- Checkstyle config: `tools/maven/checkstyle.xml`
- License headers: `./mvnw apache-rat:check`
- `verify` also runs `dependency:analyze` with `failOnWarning=true`: a new import from a transitively available artifact fails the build until the dependency is declared in the module's `pom.xml`.
- japicmp compares against `japicmp.referenceVersion` from the root `pom.xml` and only checks `@Public` API.

### Documentation

- There is no docs build in this repository. The Flink docs build (`docs/setup_docs.sh` in `apache/flink`) clones the release branch of this repository and renders `docs/content` and `docs/content.zh`.
- Option tables are hand-written HTML rows; nothing is generated from `ConfigOption` definitions.

## Repository Structure

### Modules

- `flink-connector-kafka` — The connector: `KafkaSource`, `KafkaSink`, `DynamicKafkaSource`, and the Table/SQL factories. Also publishes a test-jar with `KafkaTestEnvironment*` and `testutils`.
- `flink-sql-connector-kafka` — Shaded SQL jar; relocates `org.apache.kafka`. Bundled dependencies are listed in `src/main/resources/META-INF/NOTICE`.
- `flink-connector-kafka-e2e-tests/` — `flink-streaming-kafka-test` (the job), `flink-streaming-kafka-test-base` (shared classes), `flink-end-to-end-tests-common-kafka` (the tests).
- `flink-python` — PyFlink wrappers (`pyflink/datastream/connectors/kafka.py`) and their tests (`pyflink/datastream/connectors/tests/test_kafka.py`). Maven packaging `pom`, no Java.

### Supporting directories


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [apache/flink-connector-kafka](https://github.com/apache/flink-connector-kafka) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
