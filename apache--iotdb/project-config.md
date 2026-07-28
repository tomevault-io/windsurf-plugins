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

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Apache IoTDB is a time series database for IoT data. It uses a distributed architecture with ConfigNodes (metadata/coordination) and DataNodes (storage/query). Data is stored in TsFile columnar format (separate repo: https://github.com/apache/tsfile). Current version is 2.0.11-SNAPSHOT.

## Build Commands

```bash
# Full build (skip tests)
mvn clean package -pl distribution -am -DskipTests

# Build a specific module (e.g., datanode)
mvn clean package -pl iotdb-core/datanode -am -DskipTests

# Run unit tests for a specific module
mvn clean test -pl iotdb-core/datanode

# Run a single test class
mvn clean test -pl iotdb-core/datanode -Dtest=ClassName

# Run a single test method
mvn clean test -pl iotdb-core/datanode -Dtest=ClassName#methodName

# Build with Chinese log & error messages
mvn clean package -pl distribution -am -DskipTests -P with-zh-locale

# Format code (requires JDK 17+; auto-skipped on JDK <17)
mvn spotless:apply

# Format code in integration-test module
mvn spotless:apply -P with-integration-tests

# Check formatting without applying
mvn spotless:check
```

## Integration Tests

Integration tests live in `integration-test/` (not included in default build). They require the `with-integration-tests` profile:

```bash
# Build template-node first (needed once, or after code changes)
mvn clean package -DskipTests -pl integration-test -am -P with-integration-tests

# Run tree-model ITs (simple: 1 ConfigNode + 1 DataNode)
mvn clean verify -DskipUTs -pl integration-test -am -P with-integration-tests

# Run tree-model ITs (cluster: 1 ConfigNode + 3 DataNodes)
mvn clean verify -DskipUTs -pl integration-test -am -PClusterIT -P with-integration-tests

# Run table-model ITs (simple)
mvn clean verify -DskipUTs -pl integration-test -am -PTableSimpleIT -P with-integration-tests

# Run table-model ITs (cluster)
mvn clean verify -DskipUTs -pl integration-test -am -PTableClusterIT -P with-integration-tests

# Run a single IT class (tree-model simple, 1C1D)
mvn clean verify -DskipUTs -Dit.test=ClassName -DfailIfNoTests=false -Dfailsafe.failIfNoSpecifiedTests=false -pl integration-test -am -P with-integration-tests

# Run a single IT class (tree-model cluster, 1C3D)
mvn clean verify -DskipUTs -Dit.test=ClassName -DfailIfNoTests=false -Dfailsafe.failIfNoSpecifiedTests=false -pl integration-test -am -PClusterIT -P with-integration-tests

# Run a single IT class (table-model simple, 1C1D)
mvn clean verify -DskipUTs -Dit.test=ClassName -DfailIfNoTests=false -Dfailsafe.failIfNoSpecifiedTests=false -pl integration-test -am -PTableSimpleIT -P with-integration-tests

# Run a single IT class (table-model cluster, 1C3D)
mvn clean verify -DskipUTs -Dit.test=ClassName -DfailIfNoTests=false -Dfailsafe.failIfNoSpecifiedTests=false -pl integration-test -am -PTableClusterIT -P with-integration-tests

# Run a single test method within an IT class (use ClassName#methodName)
mvn clean verify -DskipUTs -Dit.test=ClassName#methodName -DfailIfNoTests=false -Dfailsafe.failIfNoSpecifiedTests=false -pl integration-test -am -PTableSimpleIT -P with-integration-tests
```

When verifying a new feature, only run the specific IT classes/methods that were added or modified in the current branch — do not run all ITs.

To run integration tests from IntelliJ: enable the `with-integration-tests` profile in Maven sidebar, then run test cases directly.

## Code Style

- **Apache License header**: Every new file committed to the repository MUST include the Apache License 2.0 header. Use the comment syntax appropriate for the file type (e.g., `<!-- -->` for `.md`, `/* */` for `.java`, `#` for `.properties`/`.py`). JSON files are exempt (no comment syntax). Refer to existing files in the repo for the exact format.
- **Spotless** with Google Java Format (GOOGLE style). Import order: `org.apache.iotdb`, blank, `javax`, `java`, static.
- **Checkstyle** is also configured (see `checkstyle.xml` at project root).
- Java source/target level is 17 (compiled with `maven.compiler.release=17` on JDK 17+).

## Architecture

### Node Types

- **ConfigNode** (`iotdb-core/confignode`): Manages cluster metadata, schema regions, data regions, partition tables. Coordinates via Ratis consensus.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [apache/iotdb](https://github.com/apache/iotdb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
