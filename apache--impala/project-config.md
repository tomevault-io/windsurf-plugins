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

# Apache Impala

## Agent Execution Rules

This file is optimized for AI agents. Follow these terms exactly:
- MUST: required behavior.
- SHOULD: preferred behavior unless a stronger instruction conflicts.
- MAY: optional behavior.

### Preconditions

- MUST run commands in a shell that supports `source`.
- MUST perform environment configuration before any build, test, or git command.
- MUST execute the environment configuration sequence in a single shell session.
- MUST keep the working directory at `${IMPALA_HOME}` unless a step explicitly requires
  another directory.
- SHOULD use non-interactive commands.

### Deterministic Command Policy

- MUST prefer explicit commands and explicit paths over inferred behavior.
- MUST use a complete command sequence for environment setup (see Environment
  Configuration).
- SHOULD use non-interactive git commands.
- SHOULD run the narrowest build and test scope that validates the current change.
- MAY expand scope to broader builds/tests when narrow validation fails or is insufficient.

### Failure Handling Policy

- MUST stop at the first environment setup failure and report:
  - the exact failing command
  - the key stderr/stdout message
  - the next retry command
- MUST stop and report if a required path, target, or artifact is missing.
- MUST report build/test failures with the executed command and the first actionable error.
- SHOULD retry once when failure is clearly transient (for example, network flake).
- MUST ask before destructive or irreversible operations.

### Minimal Task Playbook

1. Configure environment.
2. Identify changed scope.
3. Build only the touched component(s).
4. Run the narrowest relevant tests.
5. Expand validation only if needed.
6. Summarize exact commands run, outputs, and follow-up actions.

## Environment Configuration

Before running shell commands, configure the environment.

Required one-line sequence. If the absolute repo root path is unknown, resolve it
with `git rev-parse --show-toplevel` from any directory inside the repo:

`export IMPALA_HOME=$(git rev-parse --show-toplevel) && cd "${IMPALA_HOME}" && source "${IMPALA_HOME}/bin/impala-config.sh" && source "${IMPALA_HOME}/bin/set-classpath.sh"`

If you need stepwise execution, use this exact order:
1. `export IMPALA_HOME=$(git rev-parse --show-toplevel)`
2. `cd "${IMPALA_HOME}"`
3. `source "${IMPALA_HOME}/bin/impala-config.sh"`
4. `source "${IMPALA_HOME}/bin/set-classpath.sh"`

After sourcing, `IMPALA_BUILD_THREADS` is automatically set. Verify setup succeeded
by checking that `echo $IMPALA_HOME` prints the repo root and `echo $IMPALA_BUILD_THREADS`
prints a positive integer.

## Security

Security model: [SECURITY.md](./SECURITY.md)

Agents that scan this repository MUST consult `SECURITY.md` for threat model,
in-scope and out-of-scope declarations, and known non-findings before reporting
security issues.

## Project Overview

Apache Impala is the open source, native analytic database for open data and table
formats. It provides low latency and high concurrency for BI and analytic queries
on the Hadoop ecosystem, including Iceberg, open data formats, and most cloud
storage options. Impala also scales linearly in multitenant environments.

Impala is divided into several components:
- The Impala Daemon (`impalad`) serves as query coordinator and executor. In most
  deployments, each daemon instance is either a coordinator or an executor. Clients
  connect to coordinator instances. This daemon handles query planning and
  coordination and is implemented in C++ (query handling) and Java (query planning).
  It also manages metadata caching and retrieval from `catalogd`.
- The Impala Catalog Daemon (`catalogd`) is written mostly in Java with some C++
  for network communication. It manages metadata for all Impala daemons in a
  cluster and distributes table and database metadata to all `impalad` nodes. It
  acts as a caching layer over other catalogs (for example Hive Metastore and
  Iceberg REST catalogs).
- The Impala State Store (`statestored`) is written in C++ and tracks health and
  status of all Impala daemons in a cluster.
- The Impala Shell (`impala-shell`) is a Python command-line interface for
  connecting to and interacting with Impala daemons.

## Folder Structure

- `be`: Backend C++ codebase (query execution engine, storage engine, and low-level
  components). Uses CMake. Uses CTest and GoogleTest.
- `bin`: Utility scripts and executables for development, build, and test.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [apache/impala](https://github.com/apache/impala) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
