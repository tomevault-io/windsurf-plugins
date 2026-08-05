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

# Apache StreamPark — Agent Instructions

Project conventions, architecture, and coding patterns for the StreamPark codebase.

## Architecture

### Module Boundaries

StreamPark is a Maven multi-module project with four top-level modules. Each has a clear responsibility boundary.

- **`streampark-common`** (`streampark-common/`): Shared foundation layer. Contains configuration management (`ConfigKeys`, `ConfigOption`), utility classes (`Utils`, `HadoopUtils`, `JsonUtils`, `YarnUtils`), file system abstraction (`FsOperator`, `HdfsOperator`, `LfsOperator`), and shared enums (`FlinkDeployMode`, `ApplicationType`, etc.). Must be engine-agnostic — no Flink or Spark core API dependencies. All other modules depend on this module.

- **`streampark-flink`** (`streampark-flink/`): Flink development framework and runtime integration. Contains the core development API (`FlinkStreaming`, `FlinkTable`, `FlinkSQL`), version-specific shims layers (`streampark-flink-shims_flink-1.xx`), job submission clients (`streampark-flink-client`), Kubernetes integration (`streampark-flink-kubernetes`), application packer (`streampark-flink-packer`), and connectors. The shims proxy (`FlinkShimsProxy`) is the central mechanism for multi-version Flink support.

- **`streampark-spark`** (`streampark-spark/`): Spark development framework. Optional module, activated via `-Pspark` Maven profile. Contains `SparkStreaming`, `SparkBatch` core traits, Spark SQL client, and connectors. Follows the same lifecycle pattern as Flink modules.

- **`streampark-console`** (`streampark-console/`): Web management platform. Contains two submodules:
  - **`streampark-console-service`**: Spring Boot 2.7 backend, with `base/` (infrastructure), `core/` (business logic, controllers, services), and `system/` (authentication, user/role/team management) packages.
  - **`streampark-console-webapp`**: Vue 3 + Vite frontend, with `api/` (API layer), `views/` (pages), `components/` (reusable UI), `store/` (Pinia state).

The `streampark-common` module has the strongest stability guarantees — changes here affect all other modules. `streampark-flink/streampark-flink-core` public API (the `FlinkStreaming`, `FlinkTable` traits) should also be treated as stable — breaking changes require careful migration planning.

### High-Sensitivity Areas

- **`FlinkShimsProxy`**: The multi-version classloader isolation mechanism. Uses `ChildFirstClassLoader` to dynamically load version-specific shims JARs. Cached per Flink version. Changes here affect all Flink jobs across all versions. Never introduce static state that could leak across classloader boundaries.

- **`FlinkStreaming` / `FlinkTable` lifecycle**: The `main` -> `init` -> `ready` -> `handle` -> `destroy` lifecycle is the contract all user applications depend on. Changes to the execution order or initialization behavior can break existing applications in production.

- **`ConfigKeys` / `CommonConfig`**: Central configuration key definitions. Adding, removing, or renaming keys affects application configuration files, the console UI, and deployment scripts. Key names must remain backward-compatible.

- **`FlinkApplicationController` / `FlinkApplicationManageService` / `FlinkApplicationActionService`**: The core application management flow. Operations (start, stop, cancel, deploy) must be idempotent and handle all Flink states correctly. The `AppChangeEvent` annotation triggers state synchronization.

- **SQL parsing and validation** (`FlinkSql`, `FlinkSqlService`, `SqlConvertUtils`): SQL validation must be version-aware (Flink 1.12-1.20 have different SQL syntax). The `sql-rev.dict` file handles MySQL-to-PostgreSQL dialect conversion.

- **Kubernetes integration** (`FlinkK8sWatchController`): Uses Caffeine caches (`TrackIdCache`, `JobStatusCache`, `MetricCache`) for tracking K8s-deployed Flink jobs. Cache invalidation and TTL must be correct to avoid stale state.

- **Database schema changes**: All schema changes must have corresponding upgrade scripts in `streampark-console/.../script/upgrade/` for both MySQL and PostgreSQL. The `sql-rev.dict` file must be updated if new SQL dialect differences are introduced.

- **Authentication & Authorization**: `ShiroConfig`, `JWTUtil`, `ShiroRealm` — changes here affect all user access. The `@Permission` annotation and `PermissionAspect` enforce team-level resource isolation. Never weaken RBAC checks.

## Design Patterns


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [apache/streampark](https://github.com/apache/streampark) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
