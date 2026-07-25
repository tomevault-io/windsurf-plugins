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
# Apache Camel Quarkus - AI Agent Guidelines
Rules & guidelines for AI agents working on this codebase.

## Project Overview
Apache Camel Quarkus provides Quarkus extensions for Apache Camel components, enabling users to build integration applications with fast boot times and low memory footprints.

## What NOT to do
- Do NOT directly modify generated files under `docs/modules` or `src/main/generated` as they are automatically updated by maven
- Do NOT use dynamic class loading or reflection (impacts native compilation)
- Do NOT introduce Spring Boot or Spring unless explicitly required
- Do NOT commit changes without running `./mvnw process-resources -Pformat` first

## Technology Stack
- Java 17+
- GraalVM 25+ / Mandrel 25+
- Maven 3.9.11+ (For convenience the maven wrapper is provided in the root directory and can be invoked as `./mvnw` or `./mvnw.cmd`)
- Apache Camel 4.x, Quarkus 3.x. Check `pom.xml` for the current Camel, and Quarkus versions used.

## Repository Structure
```
extensions/          # Native-supported Camel component extensions
extensions-jvm/      # JVM-only extensions (not yet native-compatible)
extensions-core/     # Core Camel Quarkus extensions (core, yaml-dsl, etc.)
extensions-support/  # Shared support libraries used by multiple extensions
integration-tests/   # Integration tests JVM + native
integration-test-groups/  # Groupings of integration tests, primarilly for boosting CI speed & efficiency
integration-tests-jvm/   # JVM-only extension integration tests
integration-tests-support/ # Shared test utilities and test containers support
poms/bom/            # Runtime Bill of Materials
poms/bom-deployment/ # Deployment Bill of Materials
tooling/             # Maven plugin, scripts, templates, and tooling
catalog/             # Camel Quarkus catalog
docs/                # AsciiDoc documentation (Antora-based)
examples/            # Example projects (separate repo: camel-quarkus-examples)
```

## Key Files
- `pom.xml` - Project Maven dependency version properties and test container image names
- `poms/bom/pom.xml` - Runtime dependencies BOM
- `poms/bom-test/pom.xml` - Test dependencies BOM
- `tooling/scripts/test-categories.yaml` - Test categorization for GitHub Actions CI
- `docs/antora/antora.yml` - Antora documentation metadata. Note that `docs/local-build.sh` relies on an external `camel-website` repository and is not intended for self-contained local documentation generation within this project.

## Build Commands
```bash
./mvnw clean install -Dquickly                 # fast build, no tests
./mvnw clean install -T1C -Dquickly            # parallel fast build, no tests
./mvnw clean install                           # full build with JVM tests
./mvnw clean install -Dnative -Ddocker         # full build with native tests (very slow, -Ddocker can usually be omitted on MacOS)
./mvnw clean install -pl extensions/kafka -am  # single extension
./mvnw process-resources -Pformat              # format code & update metadata
```

**Tip:** Use `mvnd` (Maven Daemon) for faster builds. Use `-T1C` for parallel builds (1 thread per CPU core).

## Testing Commands
```bash
./mvnw test                              # unit tests
./mvnw verify                            # integration tests JVM mode
./mvnw verify -Dnative -Ddocker          # integration tests JVM + native mode (-Ddocker can usually be omitted on MacOS)
./mvnw test -Dtest=MyTest                # specific test
./mvnw test -pl integration-tests/kafka  # specific module
```

## Extension Structure
```
extensions/kafka/
├── pom.xml                           # Parent POM
├── runtime/
│   ├── pom.xml
│   └── src/
│       ├── main/java/org/apache/camel/quarkus/component/kafka/
│       ├── main/resources/META-INF/quarkus-extension.yaml
│       └── main/doc/                 # Extension documentation
│           ├── configuration.adoc
│           └── usage.adoc
└── deployment/
    ├── pom.xml
    └── src/
        ├── main/java/org/apache/camel/quarkus/component/kafka/deployment/
        │   └── KafkaProcessor.java
        └── test/java/
```

### Naming Conventions
Classes:
- Deployment: `*Processor.java` for build-time processing `@BuildStep`
- Runtime: `*Recorder.java` for recording and producing `RuntimeValue` used in deployment build steps
- Runtime: `*BuildTimeConfig.java` for `ConfigPhase.BUILD_TIME` & `ConfigPhase.BUILD_AND_RUN_TIME_FIXED` annotated `@ConfigMapping` interfaces

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [apache/camel-quarkus](https://github.com/apache/camel-quarkus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
