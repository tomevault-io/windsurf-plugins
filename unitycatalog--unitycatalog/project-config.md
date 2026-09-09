---
trigger: always_on
description: Operating manual for AI coding assistants working in the Unity Catalog repository, and a dense onboarding reference for humans. Most agent tools load this file automatically at the start of a session. Read it before making changes. Keep edits factual and concrete; this is a working manual, not a design document.
---

# Unity Catalog - AI Assistant Context

Operating manual for AI coding assistants working in the Unity Catalog repository, and a dense onboarding reference for humans. Most agent tools load this file automatically at the start of a session. Read it before making changes. Keep edits factual and concrete; this is a working manual, not a design document.

Unity Catalog is an open, multimodal catalog for data and AI. It is built on an OpenAPI specification with an open-source server implementation under the Apache 2.0 license, and it is interoperable with the Apache Hive metastore API and the Apache Iceberg REST catalog API. It governs access to catalogs, schemas, tables, volumes, functions, and models, so authorization and correctness mistakes have direct consequences for the data a deployment protects. Treat every change here as security- and data-sensitive by default.

## Operating principles

These are the non-negotiables. The rest of this file makes them concrete.

1. **Explore before you change.** Read the surrounding code, the module, and the existing tests first. Mirror an existing pattern instead of inventing a new one. When an analogous feature already exists (another service, repository, or connector path), follow it.
2. **Understand every line you submit.** Do not open a pull request with generated or suggested code you cannot explain and defend in review. The most common review question in this project is "why?"; if you cannot answer it for a line, do not include the line.
3. **Tests ship with the change.** New behavior and bug fixes include tests in the same pull request, in the correct module, verifying both success and failure. A logic change without a test will be sent back.
4. **Never hand-edit generated code.** API models and client code are generated from the OpenAPI spec. Edit the spec, regenerate, and commit both together. See "Spec-first: generated code."
5. **Keep pull requests small and single-purpose.** Do not mix a refactor with a feature, or bundle unrelated fixes. A refactor PR contains zero behavior changes. Split them.
6. **Preserve the public contract.** The REST API, the OpenAPI spec, released property and config names, and public client interfaces are compatibility surfaces. Do not break them casually. See "Compatibility and stability."
7. **Format, test, and self-review before requesting review.** Run the formatter and the relevant tests locally, then re-read your own diff against the conventions below and the "Common reasons a pull request gets blocked" checklist. Ask a human only after that.

## Build, test, format, generate

Unity Catalog is a JVM project, and mostly Java: the server, the Java client, the Hadoop connector, and the CLI are Java, while Scala 2.13 is used only for the sbt build and part of the Spark connector. (The Python client and the `ai/` library are Python; the UI is TypeScript.) A single JDK 17 toolchain compiles and runs all the JVM code, so install JDK 17 and point `JAVA_HOME` at it. Always use the bundled launcher `build/sbt`, not a system `sbt`. Scope a task to one module with `module/task` (for example `server/test`).

```bash
build/sbt clean compile                 # compile everything, no tests
build/sbt clean package publishLocal    # build and publish artifacts locally
build/sbt -J-Xmx2G clean test           # full test suite (give it heap)
build/sbt "server/test"                 # one module's tests
build/sbt "server/testOnly io.unitycatalog.server.service.PermissionServiceTest"  # one class (fast loop)
build/sbt -J-Xmx2G jacoco               # coverage report
build/sbt createTarball                 # distributable tarball
build/sbt javafmtAll                    # auto-fix Java formatting (run before every push)
build/sbt generate                      # regenerate OpenAPI models + client SDK (after spec edits)
```

Run the server, CLI, and UI locally:

```bash
build/sbt package
bin/start-uc-server            # starts the server on localhost
bin/uc table list           # exercise the CLI against it
cd ui && yarn install && yarn start
```

Java follows the Google Java Format (a format check runs in CI; run `build/sbt javafmtAll` before pushing). Scala follows the Apache Spark Scala Style Guide. The UI has its own format check (`yarn test:format` in `ui/`).

**CI checks a pull request must satisfy:** unit tests across an engine/format matrix (currently Spark 4.0/4.1/4.2 with the pinned Delta release, plus informational non-blocking runs against a Delta development snapshot; the primary combination also runs license and connector tests), the Java and UI format checks, the cross-Spark build, and the Python client generation and tests. The development-snapshot runs do not block, but a failure there signals an upcoming incompatibility worth understanding.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [unitycatalog/unitycatalog](https://github.com/unitycatalog/unitycatalog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
