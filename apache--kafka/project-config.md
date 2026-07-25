---
trigger: always_on
description: Licensed to the Apache Software Foundation (ASF) under one or more
---

<!--
 Licensed to the Apache Software Foundation (ASF) under one or more
 contributor license agreements.  See the NOTICE file distributed with
 this work for additional information regarding copyright ownership.
 The ASF licenses this file to You under the Apache License, Version 2.0
 (the "License"); you may not use this file except in compliance with
 the License.  You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

 Unless required by applicable law or agreed to in writing, software
 distributed under the License is distributed on an "AS IS" BASIS,
 WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 See the License for the specific language governing permissions and
 limitations under the License.
-->

# Agent Guide for Apache Kafka

This file is read by automated agents (security scanners, code analyzers,
AI assistants) operating on this repository.

## Project overview

Apache Kafka is a distributed event streaming platform. This repo is a **Gradle**
build in **Java** and **Scala 2.13**.

- **Supported JDK**: 11+ for `clients`, `generator`, and `streams`; 17+ for other modules.
- **Build/test JDK**: Java 17 and 25.
- **Build tool**: Gradle, wrapper `./gradlew` is recommended.

## Repository layout (high level)

| Path                                                                                          | Purpose                                                                                            |
|-----------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------|
| `clients/`                                                                                    | Producer, consumer, admin client APIs and protocol messages                                        |
| `core/`                                                                                       | Broker runtime, log, replication, request handling (gradually being removed along with Scala code) |
| `server/`, `server-common/`                                                                   | Broker/server components and shared server code                                                    |
| `metadata/`, `raft/`                                                                          | KRaft metadata layer and Raft implementation                                                       |
| `storage/`, `storage/api/`                                                                    | Log segments, checkpoints, tiered storage APIs                                                     |
| `group-coordinator/`, `share-coordinator/`, `transaction-coordinator/`, `coordinator-common/` | Coordinators                                                                                       |
| `connect/`                                                                                    | Kafka Connect (api, runtime, plugins, transforms)                                                  |
| `streams/`                                                                                    | Kafka Streams (+ Scala, examples, upgrade system tests)                                            |
| `generator/`                                                                                  | RPC/message code generation                                                                        |
| `tools/`, `shell/`                                                                            | CLI tools and kafka metadata shell                                                                 |
| `examples/`                                                                                   | Kafka producer and consumer examples                                                               |
| `jmh-benchmarks/`                                                                             | Kafka benchmarks tests                                                                             |
| `trogdor/`                                                                                    | test framework                                                                                     |
| `tests/`                                                                                      | System test harness (see `tests/README.md`)                                                        |
| `config/`                                                                                     | Sample broker/controller configuration                                                             |
| `checkstyle/`                                                                                 | Checkstyle rules and import-control XML                                                            |
| `docs/`                                                                                       | Documentation sources                                                                              |
| `docker/`                                                                                     | Docker images, build/test scripts, and examples for JVM and native Kafka           |


## Build commands

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [apache/kafka](https://github.com/apache/kafka) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
