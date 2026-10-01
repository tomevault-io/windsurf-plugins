---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

`fhir-tools` builds the **Ballerina Health Tool** (`bal health`) — a `bal` CLI tool for generating Ballerina
artifacts (packages, service templates, client connectors) from FHIR Implementation Guides (IGs) and CDS Hooks
definitions. It is a multi-module Maven project (Java 21) whose final output is packaged as a Ballerina tool
(`health-tool-ballerina`) and published to Ballerina Central. A separate Python MCP server exposes the same
`bal health` commands as AI-callable tools.

## Prerequisites & one-time setup

- Java 17+ (repo compiles with `maven.compiler.source/target` = 21)
- Ballerina Swan Lake **2201.12.3** (`bal` must be on `PATH`) — required to run the `bal pack`/`bal test` steps
  invoked from Maven
- A GitHub Personal Access Token with read access to GitHub Packages, added to `~/.m2/settings.xml` with server id
  `ballerina-language-repo` (needed to resolve `ballerina-lang`/`ballerina-cli` artifacts):
  ```xml
  <servers>
      <server>
          <id>ballerina-language-repo</id>
          <username>{Github_username}</username>
          <password>{Github_PAT}</password>
      </server>
  </servers>
  ```
- The [`wso2/open-healthcare-codegen-tool-framework`](https://github.com/wso2/open-healthcare-codegen-tool-framework)
  repo (currently pinned to `v2.1.2` via `version.healthcare.tool.framework` in the root `pom.xml`) must be built
  and `mvn clean install`ed locally first — this repo's modules depend on its `commons`/`fhir-core` artifacts, which
  are not on Maven Central.
- If `mvn clean install` fails resolving `org.wso2.healthcare.codegen.tool.framework:*` even after installing the
  framework above, check `~/.m2/repository/org/wso2/healthcare/codegen/tool/framework/*/*.lastUpdated` — Maven
  caches a *negative* resolution result there if it ever tried (and failed) to fetch these artifacts remotely
  before the local install existed. Delete that directory tree and rebuild; these artifacts only ever come from
  the local install, never from `ballerina-language-repo` or Central.

## Build & test

```shell
# from repo root, after the codegen-tool-framework has been installed locally
mvn clean install
```

This builds every module in the order declared in the root `pom.xml` and finishes by running `bal pack` to produce
the `health-tool-ballerina` Ballerina package under `ballerina/target/health-tool-ballerina`.

- Build a single module (respecting the reactor for its dependencies): `mvn clean install -pl native/health-cli -am`
- **Only `native/health-cli` has JUnit coverage** (JUnit 5, added alongside the `--ig` registry-download feature).
  `fhir-to-bal-template`, `fhir-to-bal-connector`, and `cds-bal-template` have no `src/test` directory and no test
  dependency at all — see "Verifying template-generation changes" below for how correctness of *those* modules is
  actually checked.
- Beyond JUnit, `native/health-cli`'s `packageGenTests` Maven profile (active by default) drives a second,
  integration-style check via `exec-maven-plugin`:
  1. `TestRunner.java` (`native/health-cli/src/test/java/TestRunner.java`) invokes the FHIR **package**-gen handler
     directly (bypassing the CLI/picocli layer) against the bundled fixtures — `profiles.USCore` for FHIR **R4**
     and `profiles.EuropeBase` for FHIR **R5** (system property `fhirVersion` selects which one runs), plus a CDS
     template run against `cds.hooks/tool-config.toml`. This never exercises template-mode generation.
  2. The generated Ballerina packages are then exercised with `bal test` against `.bal` test files copied in from
     `native/fhir-to-bal-lib/src/test/resources/ballerina.tests/{r4,r5}`.
  - To iterate on this locally: `mvn test -pl native/health-cli -am`. To change what's exercised, edit the fixtures
    under `native/health-cli/src/test/resources/profiles.*` or the `.bal` files under
    `native/fhir-to-bal-lib/src/test/resources/ballerina.tests/`.
- CI (`.github/workflows/ci.yml`) does the same thing on every PR: checks out this repo plus
  `open-healthcare-codegen-tool-framework` at the pinned tag, builds the framework, then runs `mvn clean install`
  on this repo.
- Release/publish (`.github/workflows/cd.yml`) is a manual `workflow_dispatch` (inputs: `bal_central_environment` =
  `STAGE`/`DEV`/`PROD`) that runs the same build, then `bal push`es `ballerina/target/health-tool-ballerina` to that
  Ballerina Central environment. On `PROD` it additionally cuts a GitHub Release tagged from the current `pom.xml`
  version, bumps the patch version across **all** module `pom.xml` files, and opens a "prepare for next dev cycle"
  PR back to `main`. This is the only supported publish path — there is no CLI command to unpublish or delete an
  already-published version, only `bal deprecate <org>/<name>[:<version>]` (reversible with `--undo`; does not
  delete anything, just discourages future dependency resolution onto it).

### Keeping versions in sync

The version is duplicated across `pom.xml` (root), `ballerina/pom.xml`, and every `native/*/pom.xml`. When bumping

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ballerina-platform/fhir-tools](https://github.com/ballerina-platform/fhir-tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
