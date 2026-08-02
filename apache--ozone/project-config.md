---
trigger: always_on
description: - Prefer the smallest correct change. Do not add features, abstractions, refactors, or cleanup that were not asked for.
---

# AGENTS instructions

## Working Style

- Prefer the smallest correct change. Do not add features, abstractions, refactors, or cleanup that were not asked for.
- Keep diffs surgical. Every changed line should trace back to the task.
  Do not reformat, rewrap, or rename adjacent code "while you are here".
- Match the surrounding module before introducing a new pattern.
  Reuse existing Ozone helpers, test scaffolding, and service abstractions where possible.
- Reuse existing Ozone and Ratis utilities when the surrounding code already uses them.
  Prefer extending an existing helper over duplicating logic or adding a new one-off abstraction.
- If there are multiple reasonable interpretations, state the tradeoff and ask instead of guessing.
- Do not wrap lines early just to make them look uniform. The checkstyle maximum (see `hadoop-hdds/dev-support/checkstyle/checkstyle.xml`) is 120 characters for Java. Use the full 120 characters before wrapping; never break a line that fits on one line.
- Use established Ozone vocabulary in code, docs, and PR text:
  SCM, OM, datanode, container, pipeline, volume, bucket, key, snapshot,
  Recon, FSO, OBS, and S3 Gateway.
  Avoid inventing new architecture terms unless the repo already uses them.

## Repository Snapshot

Apache Ozone is a multi-module Maven project. The root coordinates and version live in [`pom.xml`](./pom.xml).

Tech stack:

- Java 8 bytecode with JDK 21 runtime compatibility (see the `[21,]` profile in `pom.xml`)
- Maven build
- Hadoop RPC and gRPC over Protobuf
- RocksDB for persistent metadata
- Apache Ratis for replicated state
- JUnit 5 for tests

Two top-level aggregators:

- `hadoop-hdds/`: storage layer and shared infrastructure.
  Key submodules include `server-scm`, `container-service`, `framework`,
  `managed-rocksdb`, and `interface-{admin,client,server}`.
- `hadoop-ozone/`: Ozone services and clients.
  Key submodules include `ozone-manager`, `s3gateway`, `recon`, `datanode`,
  `dist`, `integration-test*`, and `ozonefs*`.

Service boundaries:

1. SCM manages containers, pipelines, and replication metadata.
2. OM manages namespace, keys, buckets, volumes, snapshots, and most user-visible metadata.
3. Datanodes serve container data and participate in Ratis pipelines.
4. Recon provides observability and derived metadata views.
5. S3 Gateway and OzoneFS expose external APIs on top of OM and HDDS services.

Cross-cutting changes often span multiple layers.
A feature or bug fix may need updates in `hadoop-hdds/interface-*`,
server-side handling, client translation code, and integration tests.

## Local Environment

- Use a JDK 21 runtime locally. Source and target compatibility remain Java 8.
- Ozone formatting conventions are shared through `.editorconfig`.
- If Maven behaves unexpectedly, check `java -version` and `mvn -version` first.

## Commands

Default local build flags:

- Use `-DskipShade -DskipRecon -DskipDocs` for iterative local work.
- Drop `-DskipShade` only when you need filesystem artifacts or tests that depend on the shaded Ozone FS jar.
- Drop `-DskipRecon` only when you are changing Recon UI or server behavior that must be built locally.
- Drop `-DskipDocs` only when you are changing docs or doc-generation logic.

Primary commands:

- Iterative full build: `mvn clean install -DskipTests -DskipShade -DskipRecon -DskipDocs`
- Full compile/verify smoke check: `mvn clean verify -DskipTests -DskipShade -DskipRecon -DskipDocs`
- Rebuild one module and its dependencies:
  `mvn -pl :ozone-manager -am install -DskipTests -DskipShade -DskipRecon -DskipDocs`
- Run one unit test class: `mvn -pl :ozone-manager test -Dtest=TestOzoneManagerLock -DskipShade -DskipRecon -DskipDocs`
- Run one unit test method:
  `mvn -pl :ozone-manager test -Dtest=TestOzoneManagerLock#testLockingOrder -DskipShade -DskipRecon -DskipDocs`
- Run one integration test class:
  `mvn -pl :ozone-integration-test test -Dtest=TestOmContainerLocationCache -DskipShade -DskipRecon`

CI-aligned local checks live under
[`hadoop-ozone/dev-support/checks/`](./hadoop-ozone/dev-support/checks/).
Prefer these when validating a change because they match CI layout and reporting:

- `./hadoop-ozone/dev-support/checks/unit.sh`
- `./hadoop-ozone/dev-support/checks/integration.sh`
- `./hadoop-ozone/dev-support/checks/checkstyle.sh`
- `./hadoop-ozone/dev-support/checks/rat.sh`
- `./hadoop-ozone/dev-support/checks/author.sh`

Notes:

- The check scripts write results under `target/<check-name>/` (or `$OUTPUT_DIR`).
- `build.sh` honors `FAIL_FAST=true`, `ITERATIONS=N`, and `OZONE_WITH_COVERAGE=true`.

### Local Cluster

- Build a runnable distribution when you need compose assets or a local tarball: `mvn -Pdist -DskipTests package`
- Start the default compose cluster from
  `hadoop-ozone/dist/target/ozone-*-SNAPSHOT/compose/ozone`:
  `OZONE_REPLICATION_FACTOR=3 ./run.sh -d`
- `.run/` contains IntelliJ run configurations for SCM, OM, Recon, datanodes, shells, S3 Gateway, and HA variants.

## Repository Structure

Key paths:

- `hadoop-hdds/interface-*`: Protobuf definitions and protocol-facing interfaces
- `hadoop-hdds/server-scm`: SCM server behavior
- `hadoop-hdds/container-service`: datanode-side container handling

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [apache/ozone](https://github.com/apache/ozone) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
