---
trigger: always_on
description: Repository guidance for AI agents and contributors working in `milvus-sdk-java`.
---

# AGENTS.md

Repository guidance for AI agents and contributors working in `milvus-sdk-java`.

## Repository layout

The repo builds two Maven artifacts plus standalone projects:

| Module | Artifact | Status | Location |
|---|---|---|---|
| sdk-core | `milvus-sdk-java` | Current development | `sdk-core/` |
| sdk-bulkwriter | `milvus-sdk-java-bulkwriter` | Current development | `sdk-bulkwriter/` |
| examples | standalone Maven project | docs/examples | `examples/` |
| benchmark | standalone Maven project | bench | `benchmark/` |
| tests | standalone legacy/TestNG projects | not in reactor | `tests/` |

Only `sdk-core` and `sdk-bulkwriter` are root reactor modules. Since v2.5.2, BulkWriter ships as a
separate artifact because it pulls in heavy Hadoop/Parquet/MinIO dependencies.

## Examples

`examples/` is a standalone Maven project showing SDK usage for both API generations. It is **not** part
of the main reactor build and runs independently:

```text
examples/
├── pom.xml                          # standalone; depends on local milvus-sdk-java + bulkwriter
└── src/main/java/io/milvus/
    ├── v1/                          # V1 SDK (MilvusServiceClient) examples
    ├── v2/                          # V2 SDK (MilvusClientV2) examples
    └── v2/bulkwriter/               # BulkWriter examples
```

Build and run (requires a Milvus at `localhost:19530`, Java 8+):

```bash
cd examples
mvn compile
mvn exec:java -Dexec.mainClass="io.milvus.v2.SimpleExample"
```

- The examples project depends on `milvus-sdk-java` and `milvus-sdk-java-bulkwriter` from the local Maven
  repository. After local SDK changes, run `mvn install -Dmaven.test.skip=true` at the repo root first.
- ~19 V1 and ~44 V2 example classes cover the common API surface; BulkWriter examples live under
  `v2/bulkwriter/`. Some examples need extra services (MinIO for BulkWriter, external tables) — check the
  header comment of each class for its prerequisites.
- Prefer `io.milvus.v2.*` for new example code, matching the V2-first development policy below.

## Tutorials

`tutorial/` is a set of beginner-oriented, standalone Maven projects for learning the SDK, mirroring the
`tutorial/` layout of the C++ and Rust SDKs. Unlike `examples/`, each tutorial depends on the **published**
`milvus-sdk-java` artifact (from Maven Central), not the local source tree, so the learner's experience
matches an application developer installing the SDK.

```text
tutorial/
├── pom.xml                    # parent POM; single source of the milvus-sdk-java version
├── 1_quickstart/
├── 2_collection/
├── 3_schema/
├── 4_index/
├── 5_dml/
├── 6_dql/
├── 7_database/
└── 8_rbac/
```

- `tutorial/pom.xml` is the aggregator/parent POM. The `milvus-sdk-java` version is declared once there
  (e.g. `<milvus.sdk.version>3.0.9</milvus.sdk.version>` in `<dependencyManagement>`) and inherited by the
  eight child modules — it must **not** be duplicated in each child `pom.xml`.
- **When releasing a new SDK version, update the `milvus-sdk-java` version in `tutorial/pom.xml`
  (`milvus.sdk.version`) so tutorials keep consuming the latest published release.**
- Each child module is runnable standalone: `cd tutorial/1_quickstart && mvn exec:java`.
- Connection settings use `MILVUS_URI` / `MILVUS_TOKEN` env vars, defaulting to
  `http://localhost:19530` / `root:Milvus` (same convention as the C++/Rust SDK tutorials).
- The source-linked examples used for SDK development and testing remain under `examples/`.

## SDK generations

Two API generations coexist inside `sdk-core`:

- **V1 (legacy, maintenance mode)** — `client/`, `param/`, `response/`, `connection/`, `exception/`.
  Entry point `io.milvus.client.MilvusServiceClient` (and `MilvusMultiServiceClient`), request params in
  `param/*`, response wrappers in `response/`. V1 is kept only for backward compatibility: it receives
  bug fixes and critical maintenance, but **no new features are added to V1**. New capabilities are
  implemented on V2 only.
- **V2 (active, long-term maintained)** — `io.milvus.v2/`. Entry point `io.milvus.v2.client.MilvusClientV2`
  (+ `MilvusClientV2Session`). Request/response DTO pattern like the C++/Rust/C# V2 SDKs: each operation is a
  `*Req` / `*Resp`, with conversions in `v2/service/<domain>/` and shared helpers in `v2/utils/`.
  `v2/client/globalcluster/` implements global-cluster topology discovery and primary swap. All new
  features, API additions, and pymilvus parity work target V2.

Cross-cutting shared code: `common/utils/cache/` (SchemaCache, CollectionTsCache), `orm/iterator/`
(V1/V2 query/search iterators), `pool/` (client pools), `common/interceptor/` (gRPC interceptors),
`common/utils/` (JSON, Float16, redaction, bloom/roaring bitmaps), `telemetry/` (client telemetry).

## Protos

- `sdk-core/src/main/milvus-proto/` is a git submodule holding `proto/*.proto`.
- Generated Java lives in `sdk-core/target/generated-sources/protobuf/{java,grpc-java}` and is produced at
  build time by `protobuf-maven-plugin`. Do not edit generated files.
- Generated classes land in `io.milvus.grpc` and `milvus.proto.*`; these are excluded from JaCoCo coverage.

## Building

```bash
git submodule update --init          # proto submodule

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [milvus-io/milvus-sdk-java](https://github.com/milvus-io/milvus-sdk-java) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
