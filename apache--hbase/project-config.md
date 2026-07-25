---
trigger: always_on
description: manages connections. This is the public API that external consumers depend on.
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

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
-->

# AGENTS.md

Apache HBase is a distributed, scalable big data store built on HDFS and cloud
object storage.

## Repo Structure

This is a multi-module Maven project. Modules live in arbitrarily nested
folders; enumerate them by searching for `pom.xml` files (excluding `target/`
directories). The root `pom.xml` defines the full reactor and build order.
Note that some directories from removed or merged modules (e.g.,
`hbase-hadoop2-compat/`, `hbase-protocol/`, `hbase-rsgroup/`) may still exist
as empty shells with only `target/` remnants. If a directory has no `pom.xml`,
it is not part of the active build.

### Client and Server

The fundamental divide in this codebase is client-side vs. server-side, with
several modules shared between them.

- `hbase-client` -- The client library. Builds RPC requests, handles retries,
  manages connections. This is the public API that external consumers depend on.
- `hbase-server` -- RegionServer and Master implementations. Processes RPCs,
  manages regions, stores data. The largest module by far.
- Shared modules like `hbase-common`, `hbase-protocol-shaded`, and
  `hbase-metrics-api` are dependencies of both sides.

When orienting on unfamiliar code, first determine which side of this divide
you are on.

### Module Roles

**Core data path:**
`hbase-client` -> `hbase-server` (via protobuf RPCs defined in
`hbase-protocol-shaded`)

**Gateways** (alternative client entry points):
`hbase-rest` (HTTP/JSON), `hbase-thrift` (Thrift RPC)

**Coprocessors** are HBase's server-side extension framework. They allow custom
code to run inside RegionServer and Master processes, with the same privileges
as the host process. The base `Coprocessor` interface lives in `hbase-client`;
observer and endpoint interfaces (`RegionObserver`, `MasterObserver`, etc.) live
in `hbase-server`. Endpoint implementations live in `hbase-endpoint`. The
built-in `AccessController` coprocessor enforces ACLs; `VisibilityController`
enforces cell-level visibility labels. Third-party coprocessors are loaded via
configuration or table schema.

**Server subsystems** (separated from hbase-server for modularity):
`hbase-balancer`, `hbase-procedure`, `hbase-replication`, `hbase-asyncfs`,
`hbase-zookeeper`, `hbase-http`

**Shared libraries:**
`hbase-common`, `hbase-metrics` + `hbase-metrics-api`, `hbase-logging`,
`hbase-hadoop-compat`

**Extensions:**
`hbase-extensions` (currently `hbase-openssl` for native TLS support)

**Storage codecs:**
`hbase-compression/*` (pluggable algorithms), `hbase-external-blockcache`

**Packaging and shading:**
`hbase-shaded/*`, `hbase-assembly*`, `hbase-resource-bundle`

**Tooling:**
`hbase-shell` (JRuby REPL), `hbase-hbtop`, `hbase-mapreduce`, `hbase-backup`,
`hbase-diagnostics`

**Build infrastructure** (ignore for code tasks):
`hbase-build-configuration`, `hbase-checkstyle`, `hbase-annotations`,
`hbase-archetypes/*`, `hbase-dev-generate-classpath`

**Testing:**
`hbase-testing-util`, `hbase-it`, `hbase-examples`

### Navigating with @InterfaceAudience

Classes are annotated with `@InterfaceAudience` to indicate their intended
consumer:

- `Public` -- Stable client API. External consumers depend on these.
- `LimitedPrivate` -- Internal API shared across modules, scoped to a named
  audience (e.g., `COPROC`, `CONFIG`, `REPLICATION`, `AUTHENTICATION`). The
  audience name tells you who is expected to call this code.
- `Private` -- Module-internal. Not API.

These annotations are the fastest way to determine whether a class is part of
the external surface or internal plumbing.

### Key Entry Points

When investigating a behavior, start from where it enters the system:

- **Client RPCs**: `RSRpcServices` (RegionServer) and `MasterRpcServices`
  (Master) handle all client-initiated RPCs. Trace from the method matching
  the RPC name.
- **REST gateway**: resource classes in `hbase-rest` map HTTP verbs to
  operations.
- **Thrift gateway**: handler classes in `hbase-thrift` map Thrift methods.
- **Coprocessor hooks**: observer interfaces (`RegionObserver`,
  `MasterObserver`, etc.) define extension points. Implementations are loaded
  via configuration or table schema.
- **Procedures**: `hbase-procedure` defines the framework; concrete procedures
  (table create, region split, etc.) live in `hbase-server`.
- **Configuration**: properties are defined in `hbase-default.xml` (in
  `hbase-common`) and overridden by operators in `hbase-site.xml`.
- **Wire format**: `.proto` files in `hbase-protocol-shaded` define every RPC

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [apache/hbase](https://github.com/apache/hbase) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
