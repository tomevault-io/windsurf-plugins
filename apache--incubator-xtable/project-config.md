---
trigger: always_on
description: - Licensed to the Apache Software Foundation (ASF) under one
---

<!--
 - Licensed to the Apache Software Foundation (ASF) under one
 - or more contributor license agreements.  See the NOTICE file
 - distributed with this work for additional information
 - regarding copyright ownership.  The ASF licenses this file
 - to you under the Apache License, Version 2.0 (the
 - "License"); you may not use this file except in compliance
 - with the License.  You may obtain a copy of the License at
 -
 -     http://www.apache.org/licenses/LICENSE-2.0
 -
 - Unless required by applicable law or agreed to in writing, software
 - distributed under the License is distributed on an "AS IS" BASIS,
 - WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 - See the License for the specific language governing permissions and
 - limitations under the License.
-->

# Apache XTable Agent Guide

This file gives repository-specific instructions for agents working in this repo.

## Scope

- Applies to the whole repository unless a deeper `AGENTS.md` is added later.

## Repo Structure

- `xtable-api`: shared public interfaces and SPI contracts.
- `xtable-core`: core conversion logic, sync flow, and common implementation code.
- `xtable-hudi-support`: Hudi-specific support modules, including shaded extensions under `xtable-hudi-support-extensions`.
- `xtable-utilities`: bundled CLI and shaded distribution jar.
- `xtable-aws`: AWS-related shaded support dependencies.
- `xtable-hive-metastore`: Hive Metastore shaded support dependencies.
- `xtable-service`: service-layer code.
- `release/scripts`: release and compliance automation, including shaded license tooling.
- `spec` and `rfc`: design docs and proposal material.
- `website`: project site content.

When changing code, prefer to work in the narrowest module that owns the behavior. If a change crosses module boundaries, verify all affected modules instead of only the top-level caller.

## Common Validation

Use Java 11 for local Maven work.

Common commands:

```bash
./mvnw test
./mvnw verify
./mvnw spotless:check
./mvnw spotless:apply
```

Prefer targeted commands while iterating:

```bash
./mvnw -pl <module[,module...]> test
./mvnw -pl <module[,module...]> verify
./mvnw -pl <module[,module...]> -Dtest=<TestClass> test
```

Test control flags wired in the root `pom.xml`:

- `-DskipTests` skips both unit and integration tests.
- `-DskipUTs` skips surefire unit tests only.
- `-DskipITs` skips failsafe integration tests only.

Validation expectations:

- For a focused code change, run the narrowest module test or verify command that covers the edited behavior.
- For shared build logic, parent POM changes, cross-module APIs, or release tooling, run broader Maven validation before finishing.
- If formatting might be affected, run `./mvnw spotless:check` and use `./mvnw spotless:apply` if needed.

## Dependency Changes

When adding, removing, or upgrading a dependency, always follow this sequence:

1. Update the relevant `pom.xml` dependency declarations.
2. If the module uses `maven-shade-plugin`, regenerate the runtime dependency tree and keep the shade `<artifactSet><includes>` list aligned to runtime dependencies only.
3. Regenerate bundled license metadata for shaded modules.
4. Run the shaded license validator.
5. Run the narrowest Maven verification needed for the changed modules.

Do not stop after updating the Maven dependency declaration alone.

## Shaded Modules

Modules with `maven-shade-plugin` must use explicit `<artifactSet><includes>` entries.

Rules:

- Includes must reflect the current `dependency:tree -Dscope=runtime` output.
- Do not include `provided` or `test` dependencies.
- Do not hand-wave transitive dependencies; if they are shaded, they must be listed explicitly.
- If a dependency upgrade changes the runtime tree, update the include list to match.

Current shaded modules include:

- `xtable-aws`
- `xtable-hive-metastore`
- `xtable-hudi-support/xtable-hudi-support-extensions`
- `xtable-utilities`

`xtable-utilities` is intentionally skipped by `release/scripts/validate_shaded_license_coverage.sh` while it is not published. Keep its bundled metadata useful for local builds, but do not treat it as release-blocking until that artifact is published.

## Required Commands For Dependency Work

Generate runtime dependency trees for changed shaded modules:

```bash
./mvnw -pl <module[,module...]> -am -DskipTests dependency:tree -Dscope=runtime -DoutputType=text -DoutputFile=target/dependency-tree-runtime.txt
```

Regenerate bundled license metadata:

```bash
python3 release/scripts/generate_shaded_license_metadata.py
```

Validate shaded dependency license coverage and ASF license-family compliance:

```bash
release/scripts/validate_shaded_license_coverage.sh
```

If only one or two modules changed, prefer targeted Maven verification:

```bash
./mvnw -pl <module[,module...]> -am -DskipTests dependency:tree -Dscope=runtime -DoutputType=text -DoutputFile=target/dependency-tree-runtime.txt
```

If broader confidence is needed, run:

```bash
./mvnw clean install -ntp -B
```

## Bundled License Metadata

Shaded modules must keep these files current:

- `src/main/resources/META-INF/LICENSE-bundled`
- `src/main/resources/META-INF/NOTICE-bundled`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [apache/incubator-xtable](https://github.com/apache/incubator-xtable) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
