---
trigger: always_on
description: Licensed to the Apache Software Foundation (ASF) under one
---

<!---
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
  limitations under the License. See accompanying LICENSE file.
-->


This file governs AI-assisted work on the Apache Hadoop Cloudstore codebase.


## Project

Cloudstore is a Java 8 / Maven diagnostics CLI bundled as a single JAR and invoked via `hadoop jar`. It sits on top of the Apache Hadoop FileSystem APIs and adds tools for troubleshooting cloud connectors (S3A, ABFS, ADL, GCS) — classpath/credential/network checks, faster listings, bandwidth probes, AWS V2 SDK probes, etc. It is intentionally not part of the Hadoop release: agile cadence, lighter testing, and the freedom to compile against multiple Hadoop versions are explicit goals.

All source lives under `org.apache.hadoop.fs.*` deliberately, so it can call package-private Hadoop APIs.

### Key Technologies

*   **Language:** Java 8
*   **Build:** Apache Maven
*   **Core Dependencies:**
    *   Apache Hadoop (various modules: `hadoop-client`, `hadoop-common`, `hadoop-cloud-storage`)
    *   AWS SDK for Java 2.x
    *   Google Cloud Storage Connector

Hadoop, AWS SDK v2, and the GCS connector are all `provided` scope — the JAR is designed to be dropped onto an existing Hadoop install.

## Build and run

See [BUILDING.md](./BUILDING.md).

Bytecode is pinned to Java 8 (compiler plugin + enforcer rule `[1.8,)`). The build itself runs fine on JDK 11+, and **needs** JDK 11+ if you invoke `spotless:apply` (palantir-java-format requires it).

```bash
mvn clean install                          # compile + unit tests + jar (default: hadoop.version=3.4.0)
mvn clean verify                           # adds: ITest*, apache-rat:check, spotless:check
mvn install -Pnext                         # next hadoop release
mvn install -Dtrunk                        # trunk p;roifle
mvn install -P snapshots-and-staging       # adds ASF staging/snapshot repos

mvn spotless:apply                         # auto-format Java to palantir-java-format
mvn org.apache.rat:apache-rat-plugin:check # license-header audit only
mvn site                                   # render src/site → target/site (fluido skin)
```

The active build wires `apache-rat-plugin`, `spotless-maven-plugin`, and `maven-failsafe-plugin` into the `verify` phase. RAT and Spotless will fail the build if a newly added file is missing the ASF header or hasn't been run through `spotless:apply`.


Releasing is manual and tag-driven — see `BUILDING.md`. Bump the version with `mvn versions:set -DnewVersion=…` *and* search-and-replace `cloudstore-<old>.jar` references; the `-SNAPSHOT` suffix is intentionally never used.

## Running Commands

The tools are executed via the `hadoop jar` command. The general syntax is:

```sh
hadoop jar target/cloudstore-1.4.jar <command> [options] <arguments...>
```

-   `<command>`: The name of the tool to run (e.g., `list`, `dux`, `storediag`).
-   `[options]`: Common options include:
    -   `-D <key=value>`: Define a Hadoop configuration property.
    -   `-xmlfile <file>`: Load a Hadoop configuration XML file.
    -   `-verbose`: Enable verbose output.
    -   `-debug`: Enable low-level debug logging for the JVM and connectors.

**Example: Listing files in an S3 bucket**

```sh
hadoop jar target/cloudstore-1.4.jar list -limit 10 s3a://my-bucket/path/
```


## Tests

JUnit 4 + AssertJ. Convention drives which runner picks up a class:

- `Test*` — unit tests, run by surefire under `mvn test` / `mvn install`.
- `ITest*` — integration tests, run by failsafe under `mvn verify` (forked, 600s timeout, `reuseForks=false`).

Single test / single method:

```bash
mvn test -Dtest=TestConstval
mvn test -Dtest=TestConstval#methodName
mvn verify -Dit.test=ITestLocalStorediagContract
```

Most tests are local; cloud-touching integration tests need credentials supplied via `src/test/resources/auth-keys.xml` (gitignored). Storediag-specific contract tests live under `org.apache.hadoop.fs.store.contract` and extend `AbstractFSContractTestBase`. The S3A contract test (`ITestS3AStorediagContract`) is gated by the `s3a-it` profile because the `hadoop-aws` test-jar is not consistently published to Maven Central.

## Architecture

### Two-layer command dispatch

Each user-facing command has two classes:

1. A short-name shim in the **default package**, e.g. `src/main/java/dux.java`. It is just `public static void main(String[] args) { ExtendedDu.main(args); }` plus a `help()` method called from `help.java`. This is what `hadoop jar … dux` resolves to — `hadoop jar` uses the bare class name as the entry point, and Java's default package makes it short.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [steveloughran/cloudstore](https://github.com/steveloughran/cloudstore) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
